# Capturing verification outputs

Run each command, copy the console output, and save it here as a `.txt` file:

| File name | Device | Command |
|-----------|--------|---------|
| `SW1_show-vlan-brief.txt` | SW1 | `show vlan brief` |
| `SW1_show-interfaces-trunk.txt` | SW1 | `show interfaces trunk` |
| `R1_show-ip-interface-brief.txt` | R1 | `show ip interface brief` |
| `R1_show-vlans.txt` | R1 | `show vlans` |
| `R1_show-ip-route-connected.txt` | R1 | `show ip route connected` |
| `PC1_ping-and-trace.txt` | PC1 | `ping 10.0.10.254`, `ping 20.0.20.1`, `trace 20.0.20.1` |

Tip: run `terminal length 0` first on R1/SW1 so output doesn't pause at `--More--`.

Delete this file once the outputs are added.
