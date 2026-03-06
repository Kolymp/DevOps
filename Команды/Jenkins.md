```
# Jenkins CLI (требует установки jenkins-cli.jar)
java -jar jenkins-cli.jar -s http://jenkins:8080/ list-jobs
java -jar jenkins-cli.jar -s http://jenkins:8080/ build <job-name>
java -jar jenkins-cli.jar -s http://jenkins:8080/ console <job-name>

# Groovy скрипты (Jenkins Script Console)
# Остановить все зависшие билды:
Jenkins.instance.getItemByFullName("<job>").getBuildByNumber(123).finish()
```