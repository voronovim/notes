```yaml
# https://docs.gitlab.com/ee/api/
# https://docs.gitlab.com/ee/ci/yaml/
# https://docs.gitlab.com/ee/ci/variables/predefined_variables.html

stages:
  - Build
  - Analyze
  - Upload
  - Deploy
  - Release
  - Package

variables:
  GRADLE_OPTS       : "-Dorg.gradle.daemon=false"
  REDMINE_URL       : "https://redmine.gpbdev.ru"
  OPENSHIFT_URL     : "https://openshift.dev.gazprombank.ru:8443"
  OPENSHIFT_PROJECT : "digital-platform"
  IMAGE_NAME        : "nexus.dev.gazprombank.ru:60001/${CI_PROJECT_NAME}:latest"

gradle:
  tags: [java]
  stage: Build
  script:
    #- gradlew build -x test
    - exit 0
  artifacts:
    paths: ["build/libs/*.jar"]
    expire_in: 1 hour

unit-tests:
  tags: [java]
  stage: Analyze
  script:
    #- gradlew test
    - exit 0
  artifacts:
    when: always
    paths: ["build/reports/tests/test/*"]
    reports:
      junit: build/test-results/test/TEST-*.xml

integration-tests:
  tags: [java]
  stage: Analyze
  script:
    #- gradlew integrationTest
    - exit 0
  artifacts:
    when: always
    paths: ["build/reports/tests/integrationTest/*"]
    reports:
      junit: build/test-results/integrationTest/TEST-*.xml

sonar-qube:
  tags: [java]
  stage: Analyze
  script:
    - exit 0

jar:
  tags: [java]
  stage: Upload
  script:
    #- gradlew publish
    - exit 0
  only:
    - tags

docker:
  tags: [java]
  stage: Upload
  script:
    #- docker build -t "${IMAGE_NAME}" .
    #- docker push "${IMAGE_NAME}"
    #- docker rmi "${IMAGE_NAME}"
    - exit 0
  only:
    - develop
    - tags

openshift:
  tags: [java]
  stage: Deploy
  script:
    #- oc login "${OPENSHIFT_URL}" --username="${OPENSHIFT_USER}" --password="${OPENSHIFT_PASS}" -n "${OPENSHIFT_PROJECT}" --insecure-skip-tls-verify
    #- oc rollout latest "dc/${CI_PROJECT_NAME}" -n "${OPENSHIFT_PROJECT}"
    - exit 0
  only:
    - develop
    - tags

.release-common:
  tags: [java]
  stage: Release
  script:
    #- gradlew release
    - exit 0
  when: manual
  allow_failure: false

release-candidate:
  extends: .release-common
  only:
    - /^release.*$/

release-official:
  extends: .release-common
  only:
    - master

# https://www.redmine.org/projects/redmine/wiki/rest_api
# Поиск будет осуществляться среди N последних коммитов согласно CI/CD -> General pipelines -> Git shallow clone
redmine:
  tags: [java]
  stage: Package
  variables:
    REDMINE_AUTH   : "X-Redmine-API-Key: ${REDMINE_TOKEN}"
    CONTENT_JSON   : "Content-Type: application/json"
    CONTENT_BINARY : "Content-Type: application/octet-stream"
  script:
    - |
      FILE_NAME=`ls build/libs | grep jar`
      [ ! -z "${FILE_NAME}" ] && echo "FILE_NAME -> ${FILE_NAME}" || (echo "Не найден собранный сервис" && exit 1)
    - |
      FILE_RESPONSE=`curl -s -k -H "${REDMINE_AUTH}" -H "${CONTENT_BINARY}" --data-binary "@${FILE_NAME}" -X POST "${REDMINE_URL}/uploads.json?filename=${FILE_NAME}"`
      [ ! -z "${FILE_RESPONSE}" ] && echo "FILE_RESPONSE -> ${FILE_RESPONSE}" || (echo "Ошибка загрузки файла" && exit 1)
    - |
      ID_RESPONSE=`curl -s -k -H "${REDMINE_AUTH}" -X GET "${REDMINE_URL}/users/current.json"`
      [ -z "${ID_RESPONSE}" ] && echo "Ошибка получения ID пользователя" && exit 1
      ID_RESPONSE_MASKED=`echo ${ID_RESPONSE} | sed -r 's/[a-f0-9]{40}/******/'`
      echo "ID_RESPONSE -> ${ID_RESPONSE_MASKED}"
    - |
      CURRENT=`git describe --tags --abbrev=0 || true`
      if [ -z "${CURRENT}" ]; then
        echo "Не существует ни одного тега для данной ветки"
        exit 1
      fi
      
      PREVIOUS=`git describe --tags --abbrev=0 "${CURRENT}^" --exclude "*-RC*" || true`
      if [ -z "${PREVIOUS}" ]; then
        echo "Предыдущий тег не существует - используем первый коммит ветки"
        PREVIOUS=`git log --max-parents=0 --pretty=format:"%H"`
      fi
      
      echo "CURRENT -> ${CURRENT}"
      echo "PREVIOUS -> ${PREVIOUS}"
      
      DIFF=`git log "${PREVIOUS}".."${CURRENT}" --date=short --pretty=format:"[%cd] %cn %s" --grep "Gradle Release Plugin" --invert-grep`
      [ -z "${DIFF}" ] && DIFF="Изменений нет"
      echo "DIFF -> ${DIFF}"
    - |
      TOKEN=`echo "${FILE_RESPONSE}" | jq -r '.upload.token'`
      USER_ID=`echo "${ID_RESPONSE}" | jq -r '.user.id'`
      PACKAGE=`basename "${FILE_NAME}" .jar`
      SUBJECT="Поставка ${PACKAGE} для ИФТ"
      DESCRIPTION=`echo "${DIFF}" | sed -z 's/\n/\\\\n/g'`
      
      ISSUE_REQUEST="{\"issue\":{\"project_id\":8,\"status_id\":1,\"priority_id\":2,\"subject\":\"${SUBJECT}\",\"description\":\"${DESCRIPTION}\",\"assigned_to_id\":${USER_ID},\"uploads\":[{\"token\":\"${TOKEN}\",\"filename\":\"${FILE_NAME}\",\"content_type\":\"application/octet-stream\"}]}}"
      echo "ISSUE_REQUEST -> ${ISSUE_REQUEST}"
      ISSUE_RESPONSE=`curl -s -k -H "${REDMINE_AUTH}" -H "${CONTENT_JSON}" -d "${ISSUE_REQUEST}" -X POST "${REDMINE_URL}/issues.json"`
      
      [ ! -z "${ISSUE_RESPONSE}" ] && echo "ISSUE_RESPONSE -> ${ISSUE_RESPONSE}" || (echo "Ошибка создания тикета" && exit 1)
    - |
      TICKET_ID=`echo "${ISSUE_RESPONSE}" | jq -r '.issue.id'`
      TICKET_MSG="Создан тикет ${REDMINE_URL}/issues/${TICKET_ID}"
      echo "TICKET_MSG -> ${TICKET_MSG}"
    - |
      SLACK_REQUEST="payload={\"text\":\"@channel\n${SUBJECT}\n\n${DESCRIPTION}\n\n${TICKET_MSG}\",\"link_names\":true}"
      echo "SLACK_REQUEST -> ${SLACK_REQUEST}"
      
      SLACK_RESPONSE=`curl -s -k --data-urlencode "${SLACK_REQUEST}" -X POST "${SLACK_NOTIFY_WEBHOOK}"`
      echo "SLACK_RESPONSE -> ${SLACK_RESPONSE}"
  only:
    - /^release.*$/
    - master

```