# on primary machine
mongo
    use admin
    db.createUser( {
        user: "siteUserAdmin",
        pwd: "visenze_i3_w",
        roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
      });
    db.createUser( {
        user: "siteRootAdmin",
        pwd: "visenze_i3_w",
        roles: [ { role: "root", db: "admin" } ]
      });
sudo service mongod stop

sudo openssl rand -base64 741 > /tmp/mongodb-keyfile

# on local machine
scp -i ~/.ssh/weardex-test.pem ubuntu@54.179.130.179:/tmp/mongodb-keyfile .
scp -i ~/.ssh/weardex-test.pem mongodb-keyfile ubuntu@54.169.238.35:/tmp/mongodb-keyfile
scp -i ~/.ssh/weardex-test.pem mongodb-keyfile ubuntu@54.169.242.29:/tmp/mongodb-keyfile

# on all machines
sudo chmod 600 /tmp/mongodb-keyfile && sudo chown mongodb:mongodb /tmp/mongodb-keyfile && sudo mv /tmp/mongodb-keyfile /etc/mongodb-keyfile

# on secondary machines
sudo vim /etc/mongod.conf
auth = true


# on all machines
sudo vim /etc/mongod.conf
    keyFile = /etc/mongodb-keyfile
    replSet = rs0
sudo service mongod restart


# on primary machine
# mongo
use admin
db.auth("siteRootAdmin", "visenze_i3_w");
rs.initiate()
rs.conf() # replace all hostames with ip
rs.add("other_machine_ip")
rs.status()

test_db = db.getSiblingDB('recognition')
test_db.addUser('weardex_r','visenze_i3_r',true)
test_db.addUser('weardex_w','visenze_i3_w')

git clone https://github.com/visenze/weardex-recognition-operation.git
cd weardex-recognition-operation/compose/rec-db
mongo -u weardex_w -p visenze_i3_w  localhost/recognition mongodb_scripts/create.account.js
mongo -u weardex_w -p visenze_i3_w  localhost/recognition mongodb_scripts/create.config.js
mongo -u weardex_w -p visenze_i3_w  localhost/recognition mongodb_scripts/create.model.js
mongo -u weardex_w -p visenze_i3_w  localhost/recognition mongodb_scripts/create.tag_group.js
