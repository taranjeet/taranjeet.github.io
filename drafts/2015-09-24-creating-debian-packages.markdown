Create debian packages for projects 


* First decide on the name of the project
	`<project_name>_<major_version>.<minor_version>-<package_revision>`

Example can be myproject.1-1

* Next create a folder by name which you have decided above and then move the whole codebase into that folder.

	`
	mkdir myproject.1-1
	# copy your current folder to new folder
	cp -a myproject/. myproject.1-1/
	# remember that your project root should be parallel to DEBIAN folder
	`
* Now create a directory named DEBIAN and then a file named control
	`
	mkdir DEBIAN
	touch DEBIAN\control
	subl control
	`
* Now paste the following into the file and replace it whereever required

	`
	Package: my-project
	Version: 1.1-1
	Section: base
	Priority: optional
	Architecture: all
	Depends: python (>=2.7), python-pip, python-virtualenv (>=1.11.4-1) 
	Maintainer: Taranjeet Singh <reachtotj@gmail.com>
	Description: Recharge Admin
	 This packages contains the source code for
	 my project.
	`
 * Now run

 	`
 	dpkg-deb --build myproject.1-1
 	`  

 
