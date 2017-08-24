# Docker  常见问题解决



## Running docker container : iptables: No chain/target/match by that name

```bash
iptables -t filter -F
iptables -t filter -X
service docker restart
```

​                                                                 

