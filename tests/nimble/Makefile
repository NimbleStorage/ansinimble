ROLE=NimbleStorage.AnsinimbleTest
ROLE_DIST=NimbleStorage.Ansinimble
ROLE_SRC=../..
ROLE_ROOT=/etc/ansible/roles
ROLE_DEST=$(ROLE_ROOT)/$(ROLE)
ROLE_DIST_DEST=$(ROLE_ROOT)/$(ROLE_DIST)
ROLE_LIMIT=all

all: clean copy tests
partnerless: clean copy nopartner
tag: clean copy tags
install: clean copy bundle
cloud: clean copy cv
compat: clean copy full

tags:
	ansible-playbook -t ${ARGS} -l $(ROLE_LIMIT) $(ROLE_DEST)/tests/nimble/test.yml

tests:
	ansible-playbook -l $(ROLE_LIMIT) $(ROLE_DEST)/tests/nimble/test.yml 

nopartner:
	ansible-playbook -l $(ROLE_LIMIT) --skip-tags=nopartner $(ROLE_DEST)/tests/nimble/test.yml 

copy:
	rm -rf $(ROLE_DEST)
	mkdir -p $(ROLE_DEST)
	cp -a $(ROLE_SRC)/* $(ROLE_DEST)/

copyhost:
	rm -rf $(ROLE_DIST_DEST)
	mkdir -p $(ROLE_DIST_DEST)
	cp -a $(ROLE_SRC)/* $(ROLE_DIST_DEST)/

clean:
	rm -rf $(ROLE_DEST)	

bundle:
	ansible-playbook -l $(ROLE_LIMIT) $(ROLE_DEST)/examples/nimble/sample_install.yml 

unbundle:
	ansible-playbook -l $(ROLE_LIMIT) $(ROLE_DEST)/examples/nimble/sample_uninstall.yml
cv:
	ansible-playbook -l $(ROLE_LIMIT) $(ROLE_DEST)/tests/nimble/cloud.yml 
full:
	sudo pip install ansible==2.3.3.0
	ansible-playbook -l $(ROLE_LIMIT) $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.4.6.0
	ansible-playbook -l $(ROLE_LIMIT) $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.5.15
	ansible-playbook -l $(ROLE_LIMIT) -b $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.6.16
	ansible-playbook -l $(ROLE_LIMIT) -b $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.7.10
	ansible-playbook -l $(ROLE_LIMIT) -b $(ROLE_DEST)/tests/nimble/test.yml 
	sudo pip install ansible==2.4.6.0
