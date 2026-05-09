
Bug: since we are using name in all the places and storing it in db, since name is changed we are not syncing up all the tables, instead we can use db lookup whenever we have to display


```
claude --dangerously-skip-permissions
```

```bash
cd ~/Downloads/claude-setup/claude-mem
npm run build-and-sync
```

* keep created dates at right side
* instead of clients, keep generic name in sidebar
* keep generic button and then to choose client register dropdown kind as shared in discord
add download file (.txt) for api key password
* keep help icon in clients page to suggest what is clients for better UX




best approach, merge all dependabots to one PR and rise to main and merge at once
fix the vulnerabilities in security and quality tab - https://github.com/finternet-io/finternet-app/security/dependabot
as of now work on vulnerabilities (critical, high, medium and low)other issues accross all the repos