ROLE=NimbleStorage.AnsinimbleTest
ROLE_SRC=../..
ROLE_DEST=/etc/ansible/roles/$(ROLE)

all: clean copy tests
partnerless: clean copy nopartner
tag: clean copy tags
install: clean copy bundle
cloud: clean copy cv
compat: clean copy full

tags:
	ansible-playbook -t ${ARGS} -l smokers $(ROLE_DEST)/tests/nimble/test.yml

tests:
	ansible-playbook -l smokers $(ROLE_DEST)/tests/nimble/test.yml 

nopartner:
	ansible-playbook -l smokers --skip-tags=nopartner $(ROLE_DEST)/tests/nimble/test.yml 

copy:
	mkdir -p $(ROLE_DEST)
	cp -a ${ROLE_SRC}/* $(ROLE_DEST)/

clean:
	rm -rf $(ROLE_DEST)	

bundle:
	ansible-playbook -l smokers $(ROLE_DEST)/examples/nimble/sample_install.yml 

unbundle:
	ansible-playbook -l smokers $(ROLE_DEST)/examples/nimble/sample_uninstall.yml
cv:
	ansible-playbook -l smokers $(ROLE_DEST)/tests/nimble/cloud.yml 
full:
	sudo pip install ansible==2.2.3.0
	ansible-playbook -l smokers $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.3.3.0
	ansible-playbook -l smokers $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.4.6.0
	ansible-playbook -l smokers $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.5.8
	ansible-playbook -l smokers -b $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.6.3
	ansible-playbook -l smokers -b $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.4.6.0
