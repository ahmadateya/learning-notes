# My notes from learning system commands in Linux

- **Boot, reboot, and shutdown a system safely**
  - **sudo systemctl poweroff**
    - **--force** = reset the system  
    - **--force --force** = unplug the power
  - **sudo systemctl reboot**
    - you can use --force to force a reboot and  if there is a problem 
  - **sudo shutdown [time]** to schedule the shutdown or reboot, [time] in format **from 00:00 to 23**

- **Default boot group**
  - **sudo systemctl get-default** to see the current default boot group
  - list the boot groups with **sudo systemctl list-unit-files --type=boot**
    - **user** = user mode
    - **system** = system mode
    - **firmware** = firmware mode
    - **graphical** = graphical mode
    - **multi-user** = multi-user mode
    - **graphical.target** = graphical mode
    - **multi-user.target** = multi-user mode
    - **firmware.target** = firmware mode
    - **user.target** = user mode
    - **system.target** = system mode
  - **sudo systemctl set-default [target]** and then reboot to see the new default boot group, or use **sudo systemctl isolate [your-boot-group]** but this won't change the default boot group
  
- one of the bootloaders job is to start the kernel and the most famous bootloaders for linux is **GRUB**  
