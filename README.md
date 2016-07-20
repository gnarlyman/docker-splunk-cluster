# Docker Compose to setup Cluster (including SHC and Indexing Cluster)

> Work in progress. Use as reference guide.

- Copy license to `./cluster-master/` folder.
- Build images `docker-compose build` (based on `outcoldman/splunk:6.4.2`)
- `docker-compose up -d cluster-master`
- Do `docker-compose logs -f cluster-master` and wait for line `Initialized cluster-master as Cluster Master`.
- Start first IDXC Slave `docker-compose up -d cluster-member` (`docker-compose` does not allow start and scale in one command).
- Scale IDXC Slaves to whatever number you want `docker-compose scale cluster-member=8`.
- Start first SHC Member `docker-compose up -d shc-member`.
- Scale SHC Members to whatever number you want `docker-compose scale shc-member=3`
- Bootstrap SHC `docker exec -it cluster_shc-member_1 entrypoint.sh splunk bootstrap shcluster-captain -auth admin:changeme -servers_list "https://3cd681e014a9:8089,https://e50d43cd78a9:8089,https://b201a0cea918:8089"` (change hostnames to container ids from `docker ps --filter=label=splunk.cluster=shc-member`).
- Open "Indexer Clustering" page on Cluster Master node and wait when replication factor will be met.

## How?

The main idea is simple, to use original `outcoldman` (or RelEng\official Splunk) images without any modifications to `entrypoint.sh`. On top build images for every role but just including few python scripts (`init.py` and some conf files). Invoke these scripts with `SPLUNK_CMD` command.

## TODO:

- [ ] Secret storage for getting secrets (currently everything is in plain text)
- [ ] Allow to configure few things using Environment Variables
- [ ] Requirement for License should be optional (so it will be easier to try)

## Bootstrap a captain example

```
docker exec -it cluster_shc-member_1 entrypoint.sh splunk bootstrap shcluster-captain -auth admin:changeme -servers_list "https://f0b64885cfa9:8089,https://c48c6a8c4d1d:8089,https://b5e331a49a11:8089"
```