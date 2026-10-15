# Lab 01: Your First Virtual Machine

## 1. Resource Justification
My host is an Apple M1 Mac running macOS. It has 8 CPU cores, 8 threads, 8 GB of RAM and 125 GB of available storage.
While idle, my host uses approximately 4.5 GB of RAM.
I allocated 2 vCPUs out of 8 threads because this provides sufficient processing power for Ubuntu Server while leaving resources available for macOS.
I allocated 2048 MB (2 GB) of my 8 GB RAM because Ubuntu Server runs without a graphical interface. I reserved 3.5 GB of RAM for the Windows guest in Lab 2.
I allocated a 25 GB dynamically allocated disk from 125 GB available because it provides enough storage for Ubuntu Server while minimizing initial disk usage. I also reserved at least 60 GB for the Windows guest in Lab 2.                                 

## 2. What I Broke and How I Confirmed It
I selected Option A: deleting /etc/hostname.
1. Checked the original hostname using `cat /etc/hostname`.
2. Deleted the file using `sudo rm /etc/hostname`.
3. Rebooted the virtual machine using `sudo reboot`.
4. Checked the result using `cat /etc/hostname` and `hostnamectl`.
After rebooting, the hostname configuration was missing, confirming that the system had been damaged.

## 3. Rollback Confirmation
I restored the virtual machine to the `clean-install` snapshot using VirtualBox Manager.
After starting the VM, I ran `cat /etc/hostname` again and confirmed that the original hostname, `lab01-server`, was restored.

## 4. What Did Not Work
During the laboratory work, I did not encounter any major technical issues. The VM installation, SSH connection, breaking procedure, and snapshot rollback were completed successfully without errors.

## 5. Evidence Files
All evidence artifacts are stored in the `./evidence/` directory:
- `vm-settings.png` — virtual machine CPU, memory and disk configuration.
- `guest-release.txt` — Ubuntu release information obtained through SSH.
- `broken.png` — evidence of the broken hostname configuration.
- `snapshot-list.png` — VirtualBox snapshot list showing `clean-install` after rollback.