## This notes has been taken in watching small series about git

- **Git pull -r**
  - It will pull the changes but will not create a *merged commit*
- **Git reset --hard Vs. Git reset --soft**
  - **-- soft** is the **default** and it will remove last commit and save the changes so your code will be the same but not tracked
  - **-- hard** will get you back to the last commit but will remove your changes completely
  - **Both** of them working on the **local repositories**
- To undo the commit that has been pushed to the **Remote Repos**
  - You can pull and fix it at your local repo and then push it to the remote repo with **--force** option
    - This is dangerous when you are working with a team because they might be have the deleted commit and pushing it back and the history will be missed up
  - **Or** use **git revert &lt;commit hash&gt;**
    - It will undo the commit by reversing the changes, removing what has been added and adding what has been removed