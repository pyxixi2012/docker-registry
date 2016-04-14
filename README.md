# docker-registry

# Create Docker private registry on Centos 7 

## 1.安装环境

	env: 	aliyun 
	user : 	yunwei
	IP ： 	101.200.89.67 ， 10.171.23.180 
	Port： 	5000
	docker version ： 1.10
	
	config env 	:
	# sudo -i && vi /etc/pki/tls/openssl.cnf
		[ v3_ca ]
		# Extensions for a typical CA
		subjectAltName = IP:101.200.89.67

	# export REGISTRY_STORAGE_PATH=/data/docker/private-registry/storage
	
	参考资料：
	1. https://github.com/docker/distribution/blob/master/docs/configuration.md
	2. http://www.tianmaying.com/tutorial/docker-registry
	3. https://docs.docker.com/registry/deploying/

## 2.自签名认证,制作证书
	$ sudo -i
	# cd 
	# mkdir -p certs
	# mkdir -p certs && openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
	### input CN，Beijing, Beijing ,zhiguoguo, IT ,101.200.89.67:5000 
	# cp /root/certs/domain.crt /etc/docker/certs.d/101.200.89.67:5000/ca.crt
	


## 3.deploy a private registry server and start up 
	# service docker restart
	
	# docker run -d --name private_registry  --restart=always  -e SETTINGS_FLAVOUR=dev  -e STORAGE_PATH=/registry-storage  -v /data/docker/private-registry/storage:/var/lib/registry     -u root     -p 5000:5000     -v /root/certs:/certs     -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt     -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key     registry:2
	
	# docker stop private_registry && docker rm -v private_registry
	
## 4.test：
	pull a hello-world image to registory server ,sudo -i
	
	# docker pull hello-world

	# docker tag hello-world 101.200.89.67:5000/hello-world:latest

	view pulled images in share volmun on host machine 
	# ls -l $REGISTRY_STORAGE_PATH
	# ls -l /data/docker/private-registry/storage/docker/registry/v2/repositories/
	total 8
	drwxr-xr-x 5 root root 4096 Apr 14 13:27 **hello-world**
	drwxr-xr-x 3 root root 4096 Apr 14 13:36 jpizarrom
			
## 5.client:
	提供docker client pull 使用
	# cp /etc/docker/certs.d/101.200.89.67:5000/ca.crt /home/yunwei/ca.crt

	###
	docker client copy crt file from server
	$ sudo scp -r -P59000 yunwei@101.200.89.67:/home/yunwei/domain.crt /etc/docker/certs.d/101.200.89.67:5000/ca.crt
	###

## Q&A
	1. docker search error or not found pulled images
	2. docker-compose up  

