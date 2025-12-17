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
