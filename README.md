This project is for the automatic VM deployment with such features on board:
1) Grafana
2) Prometheus
3) Nginx
4) Ansible
5) Python-web

########################USAGE#######################
1) git clone to your local machine
2) cd to the cloned file and run vagrant up
2.1) if you'll face an issue with /vagrant directory (it will not be mounted into the created vm), please copy Vagrant2 file and spin a vm from this file
3) in a while all services should be up and running

####################################################

All provisioning is happening via Varantfile (SHELL section).
During the VM deployment all requred config files will be copied to related Ansible roles directories.
VM will be started and required dependencies will be installed.
Later ansible will be executed and via playbook file all requred app will be deployed and configured.

As an access poin briged network is used (its ip address), it is greped and used later for the python webapp.
Basically /health is a redirection towards grafana graph's page with pre-defied parametrs

Ansible playbook contains:
1) NGINX is used as a proxy to replace default path for grafana from http://<server_ip>:3000 to http://<server_ip>/grafana
for python_app it is redirecting http://localhost:8001 to http://<server_ip>/app
2) Grafana deployment and configuration to serve with subpath, anonymous view of the provided dashboard
Dashbord and datasource are imported from pre-created files. 
3) Prometheus is used from github: prometheus itself, node_exporter and blackbox_exporter, all configs are default.
To scrape python app parametrs prometheus.conf is replaced with modified conf file.
SELinux is enabled by_default with VM, it is not managed via Ansible.

IPtables are managed via ansible:  with allowed ports:
      - "22"
      - "80"
      - "443"
      - "8080"
      - "3000"
      - "8000:9999"
 
4) python simple web server is configured and executed on port 8001 which has been proxied to http://<server_ip>/app 
for /app: on the main page there is a link (HEALTH) with Prometheus Graphs which is playing role of /health page,
below: page visitors counter is located, it is not ideal (because value is string), but it updates dynamic.
-- Some improvement has been made: webapp is not configured to be a systemd service with a checkout after start, so counter will be 2 from the beggining!

This was achieved with help of the official python prometheus-client (https://github.com/prometheus/client_python).

Things to improve:
1) SElinux ansible configuration: due to some issues with python libraries (which I was not able to resolve) ansible is not able to manage SELinux
2) Grafana defaul path is http://<server_ip>/grafana/d/pythonapp , it should be /grafana/dashboard instead. No idea how to resolve this, dashbord uid rename is kinda working
but still there is /d in the path right after /grafana
3) as a workaround on the webapp Prometheus graphs page is used with required parameters, instead of /health
http://<server_bridged_ip>:9090/graph?g0.range_input=1h&g0.expr=%20rate(server_requests_total%5B1m%5D)&g0.tab=0&g1.range_input=1h&g1.expr=rate(process_resident_memory_bytes%7Bjob%3D%27python-app%27%7D%5B1m%5D)&g1.tab=0&g2.range_input=1h&g2.expr=rate(process_cpu_seconds_total%7Bjob%3D%27python-app%27%7D%5B1m%5D)&g2.tab=0

But NGINX is not proxying it to /health

4) Definetelly there should be some check to be sure that all required services are up and running!!!
5) Ansible usage with ssh-key


* there is an additional Vagrantfile_old which does not have all the latest updates, but it might be used if they will be applied as an alternative way of automation
