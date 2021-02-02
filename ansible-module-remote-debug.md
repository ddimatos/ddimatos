# Module Remote Debugging (not localhost)
Starting with Ansible 2.1, AnsibleModule-based modules are put together as a zip
file consisting of the module file and the various python module boilerplate
inside of a wrapper script. Without the help of the wrapper script it would be
hard to debug the module, the wrapper script provides options:

* explode - will turn wrapper’s string into some python hierarchy and files
  that you can work with
* execute - invokes the exploded module on the arguments in the args
* excommunicate - imports the main function from the module and then calls that,
  this is more useful for GUI based debuggers and different from the way the
   module is executed by Ansible itself thus might not work for localized
   debugging

1. On the controller machine running Ansible, instruct it to leave the files on
   the remote host:<br>              `export ANSIBLE_KEEP_REMOTE_FILES=1`
2. Run your playbook:<br>
   `ansible-playbook -i inventories/mvs/inventory mvs.facts.yaml -vvvvv`
3. On remote host, the module path follows this pattern:<br>
   /.ansible/tmp/ansible-<unique id>/<module name or "arguments">
4. On the remote hose run commands:<br>
   `ls -la .ansible/tmp/ansible-tmp-1573066152.210413-62413336014771/AnsiballZ_setup.py`<br>
   `cd .ansible/tmp/ansible-tmp-1573066152.210413-62413336014771/`<br>
5. Unpack the compressed modules:<br>
   `python AnsiballZ_setup.py explode`<br>
6. Change directory into the exploded directory structure:<br>
   `cd debug_dir/`
7. List the contents of the directory structure, notice directory Ansible and file args:<br>
   `ls -laR`
8. Execute the module as if it had been run from the controller machine:<br>
   `python AnsiballZ_setup.py execute`
9.  From this point, you have access to the exploded files and modules. You can debug by editing the files directly
with either print or better yet, pdb debug statements and use execute to run the changes. When you are certain
a fix is in place, edit the original files on the controller and repeat your test. 

Using pdb or other debuggers is a separate topic, but the basic idea is you edit the code with:<br>
`Set a breakpoint in the module:`<br>
`import pdb; pdb.set_trace()`