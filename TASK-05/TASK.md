I found the problem. 👍

The issue is **not Apache**—it's your **iptables rule order**.

Your INPUT chain is:

```text
1  ACCEPT RELATED,ESTABLISHED
2  ACCEPT ICMP
3  ACCEPT lo
4  ACCEPT SSH
5  REJECT ALL   <-- ❌ Problem
6  ACCEPT LBR on port 3001
7  DROP port 3001
```

The packet reaches **Rule 5** first:

```text
REJECT all
```

so it **never reaches** Rule 6 (ACCEPT from LBR).

Remember:

> 🔥 **iptables follows "First Match Wins."**

---

## ✅ Fix

Delete the incorrect rules:

```bash
iptables -D INPUT 7
iptables -D INPUT 6
iptables -D INPUT 5
```

Now add them back in the correct order:

```bash
iptables -A INPUT -s 10.244.221.54 -p tcp --dport 3001 -j ACCEPT
iptables -A INPUT -p tcp --dport 3001 -j DROP
iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
```

Now check:

```bash
iptables -L INPUT --line-numbers -n
```

It should look like:

```text
1 ACCEPT RELATED,ESTABLISHED
2 ACCEPT icmp
3 ACCEPT lo
4 ACCEPT ssh
5 ACCEPT 10.244.221.54 tcp dpt:3001
6 DROP tcp dpt:3001
7 REJECT all
```

---

## Save the Rules

```bash
service iptables save
```

or

```bash
iptables-save > /etc/sysconfig/iptables
```

---

## Test Again

From **stlb01**:

```bash
curl http://stapp01:3001
```

✅ It should now work.

From **stapp02**:

```bash
curl http://stapp01:3001
```

❌ It should fail.

---

## 💡 Why This Happened (Interview Question)

**Q: Does iptables evaluate all matching rules?**

**Answer:** No. `iptables` processes rules from top to bottom and stops at the **first matching rule**. This is why rule ordering is critical. A broad `REJECT` or `DROP` placed before a more specific `ACCEPT` rule will prevent the `ACCEPT` rule from ever being reached.

This is a classic Linux firewal
l troubleshooting scenario and a favorite topic in DevOps and Linux administration interviews.

---

For your GitHub README and Task-05 documentation, I recommend updating the **Troubleshooting** and **Interview Tip** sections with the lesson we learned during debugging. This makes your repository more valuable because it documents a real production issue.

---

# 🐧 Linux Level-2

# 🔥 Task-05 - Secure Apache Port Using iptables

*(Everything up to "Troubleshooting" remains the same.)*

---

# 🚨 Troubleshooting

## ❌ "No route to host"

This is one of the most common issues while working with **iptables**.

### Possible Causes

* 🔥 Firewall rule order is incorrect.
* 🔥 A `REJECT` or `DROP` rule is evaluated before the `ACCEPT` rule.
* 🔥 Apache service is not running.
* 🔥 Apache is listening on a different port.

---

### Check Rule Order

```bash
iptables -L INPUT --line-numbers -n
```

❌ Incorrect Order

```text
1 ACCEPT RELATED,ESTABLISHED
2 ACCEPT ICMP
3 ACCEPT lo
4 ACCEPT SSH
5 REJECT all
6 ACCEPT LBR
7 DROP 3001
```

Since **iptables follows "First Match Wins"**, traffic is rejected before reaching the ACCEPT rule.

---

### Correct Order

```text
1 ACCEPT RELATED,ESTABLISHED
2 ACCEPT ICMP
3 ACCEPT lo
4 ACCEPT SSH
5 ACCEPT <LBR_IP> tcp dpt:3001
6 DROP tcp dpt:3001
7 REJECT all
```

---

### Delete Incorrect Rules

```bash
iptables -D INPUT 7
iptables -D INPUT 6
iptables -D INPUT 5
```

---

### Re-add Rules

```bash
iptables -A INPUT -s <LBR_IP> -p tcp --dport 3001 -j ACCEPT

iptables -A INPUT -p tcp --dport 3001 -j DROP

iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
```

---

### Save Rules

```bash
service iptables save
```

or

```bash
iptables-save > /etc/sysconfig/iptables
```

---

### Verify Rules

```bash
iptables -L INPUT --line-numbers -n
```

Expected:

```text
5 ACCEPT <LBR_IP> tcp dpt:3001
6 DROP tcp dpt:3001
7 REJECT all
```

---

# 💡 Production Tips

* ✅ Always place **specific ACCEPT rules before general DROP or REJECT rules**.
* ✅ Verify rule order after every firewall change using:

```bash
iptables -L INPUT --line-numbers -n
```

* ✅ Save firewall rules after making changes:

```bash
service iptables save
```

* ✅ Test connectivity from both **allowed** and **blocked** hosts before considering the task complete.

* ✅ Never delete the `RELATED,ESTABLISHED` rule, as it allows return traffic for existing connections.

---

# 🎤 Additional Interview Questions

### ❓ Why was the LBR unable to connect even though an ACCEPT rule existed?

Because **iptables evaluates rules from top to bottom** and stops at the **first matching rule**. A general `REJECT` rule placed before the specific `ACCEPT` rule caused the traffic to be rejected before it reached the intended allow rule.

---

### ❓ What does "First Match Wins" mean in iptables?

`iptables` processes packets sequentially. Once a packet matches a rule, the associated action is taken, and no further rules are evaluated. Therefore, **rule order is critical**.

---

### ❓ Difference between DROP and REJECT?

| DROP                                       | REJECT                                                                                |
| ------------------------------------------ | ------------------------------------------------------------------------------------- |
| Silently discards packets                  | Actively sends an error response                                                      |
| Client typically experiences a timeout     | Client receives an immediate error such as "Connection refused" or "No route to host" |
| Often used to hide services from attackers | Useful for troubleshooting and providing explicit feedback                            |

---

# ⭐ Key Takeaways

* ✔️ Installed and configured `iptables` on all application servers.
* ✔️ Allowed only the Load Balancer to access Apache on the specified port.
* ✔️ Blocked all other incoming connections to the Apache port.
* ✔️ Ensured firewall rules persist across reboots.
* ✔️ Learned that **iptables processes rules in order (First Match Wins)**.
* ✔️ Resolved a real-world firewall issue caused by an incorrectly placed `REJECT` rule.

---

This update documents the real issue you encountered during the lab, making your GitHub repository more practical and interview-focused rather than just listing the commands that were run.
