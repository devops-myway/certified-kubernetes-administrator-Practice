##### Why SSH Beats HTTPS for Git
- No passwords — uses your private key file
- One-time setup — add key once, push forever
- No token management — GitHub PATs expire and need regeneration
- Multiple accounts — easy to manage with ~/.ssh/config
- Works behind proxies

##### Steps to Switch (Using Your Existing id_rsa)
- You already have the key pair. Here's the minimal path:
- Add your public key to GitHub
- Go to GitHub → Settings → SSH and GPG keys → New SSH key
``````sh
ls -lart ~/.ssh
cat ~/.ssh/id_rsa.pub
# Copy the entire output

``````
2. Switch your remote from HTTPS to SSH
``````sh
git remote set-url origin git@github.com:devops-myway/certified-kubernetes-administrator-Practice.git
git remote -v
# Should show: git@github.com:devops-myway/... (fetch/push)

``````
3. Test GitHub SSH connection
``````sh
ssh -T git@github.com
# Expected: Hi <username>! You've successfully authenticated...
git push
``````