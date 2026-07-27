# MikroTik CLI Navigation Cheat Sheet

### Moving Between Menus

| Command      | Description |
| ------------ | ----------- |
| `/`          | Go to the **top-level root menu** |
| `..`         | Move **up one level** in the menu |
| `?`          | Show **available commands** in the current menu |
| `print`      | Show items in the current menu (e.g., interfaces, addresses, firewall rules) |

### Accessing Submenus

| Command                       | Description |
| ------------------------------ | ----------- |
| `/interface`                   | Enter interface configuration menu |
| `/interface bridge`            | Enter bridge menu |
| `/interface bridge port`       | Enter bridge port menu |
| `/interface bridge vlan`       | Enter bridge VLAN menu |
| `/ip address`                  | Enter IP address configuration menu |
| `/ip firewall filter`          | Enter firewall filter menu |
| `/ip route`                    | Enter routing table menu |
| `/system identity`             | Enter system identity configuration |

### Editing Items

| Command Example                                | Description |
| ---------------------------------------------- | ----------- |
| `add ...`                                      | Add a new item (address, interface, firewall rule, etc.) |
| `set [find ...] ...`                           | Modify existing items |
| `remove [find ...]`                            | Delete an item |

### Tips

- Typing `/` at any point takes you back to **top-level**.
- Typing `..` repeatedly moves you **up step by step**.
- Use `?` frequently to see available options and avoid syntax errors.

**Extra Tip:** For safety, always check which menu you are in by looking at the prompt. It shows your current path, e.g.:

```
[admin@MikroTik /interface bridge port]>
```
