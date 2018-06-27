ROLE=NimbleStorage.AnsinimbleTest
ROLE_SRC=../..
ROLE_DEST=/etc/ansible/roles/$(ROLE)
EC2PY=https://github.com/ansible/ansible/raw/devel/contrib/inventory/ec2.py
EC2TMP=/tmp/ec2.py
EC2TAG=tag_smokers

all: cleaner copy prepare tests
test: clean copy tests
compat: cleaner copy prepare full

tests:
	ansible-playbook -i ${EC2TMP} ${ROLE_DEST}/tests/cloud/test.yml \
	-u ubuntu \
	-l ${EC2TAG} \
	-e local_smokers=${EC2TAG} \
	-e local_role=${ROLE}

copy:
	sudo mkdir -p ${ROLE_DEST}
	sudo cp -a ${ROLE_SRC} ${ROLE_DEST}/

clean:
	sudo rm -rf ${ROLE_DEST}

cleaner:
	sudo rm -rf ${ROLE_DEST} ${EC2TMP}

prepare:
	wget -O ${EC2TMP} ${EC2PY}
	chmod 755 ${EC2TMP}

full:
	sudo pip install ansible==2.2.3.0
	ansible-playbook -i ${EC2TMP} ${ROLE_DEST}/tests/cloud/test.yml \
	-u ubuntu \
	-l ${EC2TAG} \
	-e local_smokers=${EC2TAG} \
	-e local_role=${ROLE}
	sudo pip install ansible==2.3.3.0
	ansible-playbook -i ${EC2TMP} ${ROLE_DEST}/tests/cloud/test.yml \
	-u ubuntu \
	-l ${EC2TAG} \
	-e local_smokers=${EC2TAG} \
	-e local_role=${ROLE}
	sudo pip install ansible==2.4.5.0
	ansible-playbook -i ${EC2TMP} ${ROLE_DEST}/tests/cloud/test.yml \
	-u ubuntu \
	-l ${EC2TAG} \
	-e local_smokers=${EC2TAG} \
	-e local_role=${ROLE}
	sudo pip install ansible==2.5.5
	ansible-playbook -b -i ${EC2TMP} ${ROLE_DEST}/tests/cloud/test.yml \
	-u ubuntu \
	-l ${EC2TAG} \
	-e local_smokers=${EC2TAG} \
	-e local_role=${ROLE}
	sudo pip install ansible==2.4.5.0
