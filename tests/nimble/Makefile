NLT_INSTALLER=/vagrant/nlt_installer_2.1.0.3
ROLE_SRC=../../ansinimble
ROLE_DEST=/etc/ansible/roles/NimbleStorage.Ansinimble

all: clean copy tests

tests:
	ansible-playbook -i $(ROLE_DEST)/tests/inventory $(ROLE_DEST)/tests/test.yml -e nimble_linux_toolkit_bundle=$(NLT_INSTALLER)

copy:
	mkdir -p $(ROLE_DEST)
	cp -a ${ROLE_SRC}/* $(ROLE_DEST)/

clean:
	rm -rf $(ROLE_DEST)	

install:
	ansible-playbook -i $(ROLE_DEST)/tests/inventory $(ROLE_DEST)/examples/sample_install.yml -e nimble_linux_toolkit_bundle=$(NLT_INSTALLER)
