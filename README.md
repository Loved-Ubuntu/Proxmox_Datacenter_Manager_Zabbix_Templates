# Zabbix templates for Proxmox Datacenter Manager

Two Zabbix 7.0+ templates that monitor through the Proxmox Datacenter Manager (PDM) API:

| Template | File | Monitors |
|---|---|---|
| Proxmox Datacenter Manager self by HTTP | `proxmox_datacenter_manager_self_http.json` | The PDM appliance itself: CPU, memory, swap, root disk, load, network, certificate expiry |
| Proxmox Datacenter Manager remotes by HTTP | `proxmox_datacenter_manager_remotes_http.json` | Everything PDM manages: PVE nodes, VMs, containers and storage, PBS servers and datastores |

The remotes template only talks to PDM, so Zabbix needs no access to the hypervisors or backup servers.

## Setup

1. **Create a user and API token in PDM**, for example user `zabbix@pdm` with token `monitoring`.
2. **Grant permissions** to both the user and the token (tokens are privilege-separated):
   - Role `Auditor` on `/system/status` for the self template
   - Role `Auditor` on `/resource` for the remotes template
3. **Import** the JSON file(s) in Zabbix under *Data collection → Templates → Import*.
4. **Create a host** for PDM (no interface needed) and link one or both templates.
5. **Set the host macros:**

   | Macro | Example |
   |---|---|
   | `{$PDM.URL}` | `https://pdm.example.net:8443` |
   | `{$PDM.TOKEN.ID}` | `zabbix@pdm!monitoring` |
   | `{$PDM.TOKEN.SECRET}` | the token secret |

All other macros (thresholds, intervals, discovery filters) have sensible defaults and are described in the template.

## Notes

- **TLS:** certificate verification is off by default, so PDM's self-signed certificate works out of the box. To enforce it, install the PDM CA on the Zabbix server or proxy and enable *SSL verify peer* and *SSL verify host* on the HTTP agent items.
- **Per-object thresholds:** most remotes-template macros accept a context, e.g. `{$PDM.STORAGE.PUSED.WARN:"cluster1/ceph"}` or `{$PDM.GUEST.STOP.ALERT:"cluster1/105"}=0` to silence one guest.
- **Not covered:** backup job results and replication.

## Ceph, SMART and host certificates

PDM's API does not expose Ceph health, disk SMART data or the certificates of the PVE/PBS hosts. For those, install **Zabbix agent 2** on the PVE/PBS hosts and link the official Zabbix templates:

| What | Template |
|---|---|
| Ceph health | Ceph by Zabbix agent 2 (on one or more PVE nodes) |
| Disk SMART | SMART by Zabbix agent 2 (on every PVE/PBS host) |
| Host certificate expiry | Website certificate by Zabbix agent 2 |

## License

Licensed under the [GNU General Public License v3.0](LICENSE). You are free to use, modify and share these templates for any purpose, including commercially. If you distribute a modified version, it must also be released under GPLv3 with its source.
