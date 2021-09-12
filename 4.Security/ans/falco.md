``` sh
# worker
curl -s https://falco.org/repo/falcosecurity-3672BA8F.asc | apt-key add -
echo "deb https://download.falco.org/packages/deb stable main" | tee -a /etc/apt/sources.list.d/falcosecurity.list
apt-get update -y
apt-get -y install linux-headers-$(uname -r)
apt-get install -y falco

cat /etc/falco/falco.yaml 
> syslog_output:
>  enabled: true

cat <<EOF > /etc/falco/falco_rules.local.yaml
- rule: Launch Package Management Process in Container
  desc: Package management process ran inside container
  condition: >
    spawned_process
    and container
    and user.name != "_apt"
    and package_mgmt_procs
    and not package_mgmt_ancestor_procs
    and not user_known_package_manager_in_container
  output: >
    Package management process launched in container (%evt.time , %k8s.pod.name , %container.name)
  priority: ERROR
  tags: [process, mitre_persistence]
EOF

# master
kubectl run nginx --image=nginx
kubectl exec nginx -- apt-get

# worker
cat /var/log/syslog | grep falco

# master
kubectl delete pod nginx

# worker
systemctl stop falco

```
