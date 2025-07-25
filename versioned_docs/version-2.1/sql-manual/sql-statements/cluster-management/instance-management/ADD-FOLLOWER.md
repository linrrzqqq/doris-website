---
{
    "title": "ADD FOLLOWER",
    "language": "en"
}
---

## Description

This statement adds a node with the FOLLOWER role to the FRONTEND node (only used by administrators!)

## Syntax

```sql
ALTER SYSTEM ADD FOLLOWER "<follower_host>:<edit_log_port>"
```

## Required Parameters

**1. `<follower_host>`**

> Can be the hostname or IP address of the FE node

**2. `<edit_log_port>`**

> bdbje communication port of FE node, the default is 9010

## Access Control Requirements

The user executing this SQL command must have at least the following permissions:

| Privilege | Object | Notes |
|-----------|----|-------|
| NODE_PRIV |    |       |

## Usage Notes

1. Before adding a new FOLLOWER node, make sure the node is properly configured.
2. Before adding a FOLLOWER node, ensure that the number of FOLLOWER nodes in the cluster is an odd number after the addition.
3. After adding the FOLLOWER node, use[`SHOW FRONTENDS`](./SHOW-FRONTENDS.md)command to verify that they have been added successfully and are in a normal state.

## Examples

1. Add a FOLLOWER node

   ```sql
   ALTER SYSTEM ADD FOLLOWER "host_ip:9010"
   ```
   This command adds a FOLLOWER node to the cluster (IP host_ip, port 9010)
