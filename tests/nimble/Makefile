ROLE=NimbleStorage.AnsinimbleTest
ROLE_SRC=../..
ROLE_DEST=/etc/ansible/roles/$(ROLE)

all: clean copy tests
tag: clean copy tags
install: clean copy bundle

tags:
	ansible-playbook -t ${ARGS} -l smokers $(ROLE_DEST)/tests/nimble/test.yml

tests:
	ansible-playbook -l smokers $(ROLE_DEST)/tests/nimble/test.yml 

copy:
	mkdir -p $(ROLE_DEST)
	cp -a ${ROLE_SRC}/* $(ROLE_DEST)/

clean:
	rm -rf $(ROLE_DEST)	

bundle:
	ansible-playbook -l smokers $(ROLE_DEST)/examples/nimble/sample_install.yml 

unbundle:
	ansible-playbook -l smokers $(ROLE_DEST)/examples/nimble/sample_uninstall.yml
