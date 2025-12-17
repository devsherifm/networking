#### Below is the lab connectivity running in container lab

```bash
(pytest) kali@ubuntu:~/clab/vrnetlab/cisco/iol$ clab inspect -t cisco_iol_intr_connect.clab.yml
15:24:44 INFO Parsing & checking topology file=cisco_iol_intr_connect.clab.yml
╭─────────────────────────┬────────────────────────────────┬─────────┬────────────────╮
│           Name          │           Kind/Image           │  State  │ IPv4/6 Address │
├─────────────────────────┼────────────────────────────────┼─────────┼────────────────┤
│ clab-multicast_01-iol-1 │ cisco_iol                      │ running │ 192.168.100.31 │
│                         │ vrnetlab/cisco_iol:17.12.01    │         │ N/A            │
├─────────────────────────┼────────────────────────────────┼─────────┼────────────────┤
│ clab-multicast_01-iol-2 │ cisco_iol                      │ running │ 192.168.100.32 │
│                         │ vrnetlab/cisco_iol:17.12.01    │         │ N/A            │
├─────────────────────────┼────────────────────────────────┼─────────┼────────────────┤
│ clab-multicast_01-iol-3 │ cisco_iol                      │ running │ 192.168.100.40 │
│                         │ vrnetlab/cisco_iol:l2-17.12.01 │         │ N/A            │
╰─────────────────────────┴────────────────────────────────┴─────────┴────────────────╯
```

## LAB Set up

Please refer my linkedin post for more infomration <br>
[Visit My_Linkedin](https://in.linkedin.com/in/mohammed-sherif-m)

### use below command to deply the toplogy

```bash
clab deploy -t cisco_iol_intr_connect.clab.yml
```

### use below command to view the toplogy info

```bash
clab inspect -t cisco_iol_intr_connect.clab.yml
```

### use below command to view the toplogy diagram in broswer

```bash
(pytest) kali@ubuntu:~/clab/vrnetlab/cisco/iol$ clab graph -t cisco_iol_intr_connect.clab.yml
15:31:39 INFO Parsing & checking topology file=cisco_iol_intr_connect.clab.yml
15:31:39 INFO Serving topology graph
  addresses=
  │   http://10.0.xx.15:50080
  │   http://192.168.xx.101:50080
  │   http://192.168.xx.1:50080
  │   http://172.xx.xx.1:50080
```


