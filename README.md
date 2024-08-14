# Kubernetes1.24-CIS
CIS Baseline Ansible Role for Kubernetes CIS Benchmark v1.24.
This role is designed according to the CIS Kubernetes Benchmark v1.24 policies, available at: CIS Security - Kubernetes Benchmark.
It includes an Ansible playbook with a role to remediate a Kubernetes cluster based on the CIS_Kubernetes_Benchmark_v1.24 defined on https://www.cisecurity.org/benchmark/kubernetes

For reference, this is an updated version of the previous project: Kubernetes1.6.1-CIS. https://github.com/ansible-lockdown/Kubernetes1.6.1-CIS

In the new version, I've attempted to automate all advanced CIS policies, which were not included in the previous version.

Update: 13 Aug 2024
Release note: Policy remediation in Section 5 is now available for (K8S V1.30.0 and above) using ValidatingAdmissionPolicy. There is no point in using deprecated feature PSP

Requirements
------------
Kubernetes Cluster v1.28.0 and above. Other versions have not been tested but may work. The latest version verified is Kubernetes Cluster v1.30.1.


How to use?
------------
1. Edit ansible.cfg file and define inventory = ( modify the sample inventory file provided in the role)

    The inventory exmaple below has 3 master/control plane nodes and 3 worker nodes.
   ```
    # cat ./inventories/prod/hosts
    master:
      hosts:
        master-node01.example.local:
        master-node02.example.local:
        master-node03.example.local:
    worker:
      hosts:
        worker-node01.example.local:
        worker-node02.example.local:
        worker-node03.example.local:
    ```

3. Verify the defaults/main.yml and make sure all files required by playbook does exists.
   Play is executed in the 5 CIS sections, enable/disable them at section level
   ```
   section1_patch: yes
   section2_patch: yes
   section3_patch: yes
   section4_patch: yes
   section5_patch: yes   # NOTE: implementation of ValidatingAdmissionPolicy only tested in apiVersion: admissionregistration.k8s.io/v1 which requires Kubernetes v1.30.0 and above. You may try alpha/beta in lower version.
    ```

4. You can disable individual policy rule by setting true or false
   ```
   rule_1_1_1: true/false
   ```

5. Play execution
    ```
   # Execution on master/control plane nodes
   ansible-playbook site.yml -l master --tags=master -e k8shosts=master -i ./inventories/prod/hosts
   
   # Execution on worker nodes
   ansible-playbook site.yml -l worker --tags=worker -e k8shosts=worker -i ./inventories/prod/hosts

   # If you want to run indvidual rules then,
   mv tasks/main.yml tasks/main.yml.bak
   mv tasks/main-gen.yml tasks/main.yml
   ansible-playbook site-gen.yml -l master --tags=rule_1.1.1 -i ./inventories/prod/hosts
   ansible-playbook site-gen.yml -l master --tags="rule_1.1.1,rule_1.1.2" -i ./inventories/prod/hosts
    ```
6. The ansible role is idempotent and you can run multiple times on failure.
   
7. Depending upon your system, you may need to retsart the kubelet at the end of play execution : systemctl restart kubelet
   

