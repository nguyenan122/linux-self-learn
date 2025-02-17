### Vim config fast Ansible
```
# vim ~/.vimrc 
set ts=2 sw=2
set tabstop=2
# mkdir roles
```
### Makefile
```
.PHONY: help
help: ## Show this help
	@echo "Xin Chao"
	@echo "De chay: #make install"
install:
	ansible-playbook -i inventory.yml install.yml
```

### Inventory
```
[master]
worker-node1 ansible_ssh_host=192.168.88.12 ansible_ssh_private_key_file=/home/tuanda/.ssh/id_rsa ansible_user=tuanda
[worker]
worker-node1 ansible_ssh_host=192.168.88.13 ansible_ssh_private_key_file=/home/tuanda/.ssh/id_rsa ansible_user=tuanda
worker-node2 ansible_ssh_host=192.168.88.14 ansible_ssh_private_key_file=/home/tuanda/.ssh/id_rsa ansible_user=tuanda
```

### Roles
Khai báo roles
```
---
- name: Install redis
  hosts: redis
  vars:
    - ahihi: 'kaka'
  
  roles:
    - role: redis-all-in-one
      become: true
```

### Folder and File
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html 
```
- name: create folder
  file:
    path: /setup
    state: directory #hoặc file
    owner: foo
    group: foo
    mode: '0755'
    
- name: delete folder
  file:
    path: /setup
    state: absent
```

### Debug
```
- name: show echo
  shell: echo {{ tuanda }}
  register: kiemtra

- name: Debug kiem tra lan 1
  debug:
    var: kiemtra

- name: debug kiem tra lan 2
  debug:
    msg: XIN CHAO {{ kiemtra }}
```

### Tags
Để gọi riêng từng nhóm tags, đỡ phải chạy task không cần thiết như make redis (bỏ). Ta gọi thẳng start redis

### Variable Priority
https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#understanding-variable-precedence
cái nào xuất hiện trước, sẽ được ưu tiên lấy ra đầu tiên
```
1. File trong yml role > inventory /etc/ansible/hosts
- option -e cao nhất (0)
- Khai báo trong thư mục roles/xxx/vars (1) sẽ có độ ưu tiên cao nhất, dù khai báo 2,3,4 đều không lấy biến đó.
- file role tổng (2) (file gọi các roles)
- file inventory /etc/ansible/hosts (3)
- file trong roles/xxx/default (4)
```

### group_vars và host_vars (Thư mục)
Thứ tự các file sắp xếp như dưới đấy. Việc oder file nào trước nằm ở mục Variable Priority
```
[tuanda@master-node nfs-server-client]$ cat inventory.yml 
[nfs_server]
pc1 ansible_ssh_host=192.168.56.12
[nfs_client]
pc2 ansible_ssh_host=192.168.56.13

[tuanda@master-node nfs-server-client]$ tree
.
├── group_vars
│   ├── all.yml
│   ├── nfs_client
│   │   ├── nfs_client.yml
│   │   └── nfs_server.yml
│   └── nfs_server
├── host_vars
│   ├── pc1.yml
│   └── pc2.yml
├── inventory.yml
├── nfs.yml
└── roles

```

### Copy
```
- name: Upload package require
  copy:
    src: "{{ item }}.rpm"
    dest: /setup/
  loop: "{{ package_names }}"
```

### Local Copy
Lưu file từ stdout > Ansible master
```
- name: Test x
  shell: "echo XINCHAO"
  register: x
- name: "Storing Token"
  local_action: copy content={{ x.stdout }} dest=/tmp/token
Kết quả: $ cat /tmp/token 
XINCHAO
  ```

### Copy file từ remote về Ansible Master
```
- name: Copy file from remote to ansible
  fetch:
    src: /tmp/test.txt
    dest: /tmp/download/
 > Kết Quả:
[tuanda@master-node ansible]$ cat /tmp/download/worker-node1/tmp/test.txt 
1313
[tuanda@master-node ansible]$ cat /tmp/download/worker-node2/tmp/test.txt 
1414

```

### Datetime
```
- name: Test date_time
  shell: "echo {{ ansible_date_time.year }}{{ ansible_date_time.month }}{{ ansible_date_time.day }}_{{ ansible_date_time.hour }}{{ ansible_date_time.minute }}{{ ansible_date_time.second }} > /tmp/test.txt"
  ```

### Template

### Loop
Ngoài loop theo item liệt kê. ta còn có thể loop theo variable theo loop.
```
loop:
  - {{ list_port }}
```

### Handler:
```
- name: "restart redis 1"
  service:
    name: "{{ redis_service_name }}"
    state: restarted
 Sau đó gọi bằng notify: restart redis 1
```

### Force handler running
```
- name: Force handler run
  meta: flush_handlers
```

### Create user if not exist
```
- name: check if redis user exists (ignore errors)
  command: id {{ redis_user }}
  ignore_errors: yes
  changed_when: false
  register: user_exists

- name: add redis group
  group:
    name: "{{ redis_group }}"
    state: present
  when: user_exists is failed

- name: add redis user
  user:
    name: "{{ redis_user }}"
    group: "{{ redis_group }}"
    comment: "Redis"
    home: "{{ redis_install_dir }}"
    shell: /bin/false
    system: yes
  when: user_exists is failed
  ```

### when
VD1: Compare variable string
```
[redis_server] < Ta đặt inventory variable như sau:
192.168.122.129 install_redis=yes  redis_role=master
... 
when:
    - redis_role  == 'master'
```
 Ví dụ 2: When with variable bool
 ```
 Ta đặt default var là redis_tarball: false
   when: not redis_tarball
   when: redis_tarball
Nếu là false thì
nếu là string hoặc interger thì: 
```

### For jinja
Link kiểm tra Jinja https://cryptic-cliffs-32040.herokuapp.com/
File default:
```
redis_save:
  - 900 1
  - 300 10
  - 60 10000
```
File template
```
{% for save in redis_save -%}
save {{ save }}
{% endfor -%}
```


### IF ijnja
Trong file default mặc định để false. Nếu thay đổi thì xóa false đi và thay bằng string
```
redis_bind: false
```
Trong file Template
```
{% if redis_bind -%}
bind {{ redis_bind }}
{% else %}
bind 127.0.0.1
{% endif -%}
>>>> Hoặc, để bỏ dòng bind nếu false <<<<<
{% if redis_bind -%}
bind {{ redis_bind }}
{% endif -%}

```
Giải thích: Nếu false thì sẽ không có trong file config, nếu không false thì sẽ là string và kết quả là: requirepass matkhau


### Ansible Vault
Phần 1: Tạo và chạy Vault (mã hóa all)
```
ansible-vault create test1.yaml
---
- hosts: api
  become: true
        
  tasks:
    - name: echo text
      shell: echo XINCHAO

ansible-vault edit test1.yaml
ansible-vault view test1.yaml
ansible-playbook test1.yaml --ask-vault-pass
```

Phần 2: Chèn vault vào variable
```
ansible-vault encrypt_string --vault-id @prompt thisismysupersecretstring
---
- hosts: api
  become: true

  vars:
    secret: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          66666435303938323065643534376535633535353766323062633566313038313839343038336237
          3966616433326432643365623136383332313733646333390a333762663236393636613232663762
          33313861656264313061363265653031366538373662336132626465613839313335613862343464
          6436303837643761630a666532623532616431303632613466646661326566326332306264323236
          61383331333833333530373934356333653962396661336336653034306339306333        
  tasks:
    - name: echo text
      shell: echo {{ secret }}

Sau đó ta chạy
ansible-playbook test1_encrypt.yaml --ask-vauld-pass
ansible-playbook test1_encrypt.yaml --vault-password-file=pass.txt
```