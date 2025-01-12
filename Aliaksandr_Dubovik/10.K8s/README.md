# PODS K3s yaml

```yaml
name: k3s pods

on: 
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
#  push:
#  schedule:
 #   - cron: "*/15 * * * *"
env:
  report: report.log
  crash_report: crash_report.log

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Connect to K3S host
        id: get_pods
        run: |
           which ssh-agent
           mkdir ~/.ssh
           eval `ssh-agent -s`
           ssh-add - <<< "${{ secrets.SSH_KEY }}" 
           ssh-keyscan -H ${{ secrets.BASTION_IP }} >> ~/.ssh/known_hosts
           ssh ${{ secrets.BASTION_UN }}@${{ secrets.BASTION_IP}} ssh-keyscan ${{ secrets.K3SHOST_IP }} >> ~/.ssh/known_hosts
           ssh -o ProxyCommand="ssh -W %h:%p ${{ secrets.BASTION_UN }}@${{ secrets.BASTION_IP }}" ${{ secrets.K3SHOST_UN }}@${{ secrets.K3SHOST_IP }} "kubectl get pods -A" >> report  
      
      - name: check_crash_pods
        id: check_pods 
        run: |
           echo "$(cat  report | grep -v Running)" >> crash_report 2>&1
           echo "pods_fail=$(cat report | grep -v Running | wc -l)" >> $GITHUB_OUTPUT
      
      - name: upload crashed pod's list
        if: steps.check_pods.outputs.pods_fail > 1
        uses: actions/upload-artifact@v2
        with:
          path: crash_report
      
      - name: slack Notification
        if: steps.check_pods.outputs.pods_fail > 1
        uses: rtCamp/action-slack-notify@v2 
        env:
          SLACK_CHANNEL: k8s_pods
          SLACK_TITLE: 'CHECK PODS'
          LACK_ICON: https://github.com/rtCamp.png?size=48
          SLACK_MESSAGE: 'Something happend! Check k3S!'
          SLACK_USERNAME: ntBot
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

[GitHub Actions](https://github.com/AliaksandrDub/k8s_pods/actions)