# Security Group

## References

* [Template](./../templates/bbb-on-aws-securitygroups.template.yaml)

| Resource | Usage | Values
| ------------- | ------------- | ------------- |
| SG for BBB | refer section below | |
| SG for turn | refer section below | |


### SG for BBB (Inbound)

| Ports | Protocol | 
| ------------- | ------------- |
| 16384 - 32768 | UDP | 
| 80 | TCP |
| 443 | TCP |
| 49152-65535 | UDP | 


### SG for Turn (Inbound)

| Ports | Protocol | 
| ------------- | ------------- |
| 3478 | UDP | 
| 80 | TCP |
| 443 | UDP. TCP |
| 3478 | TCP | 
