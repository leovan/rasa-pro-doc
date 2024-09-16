# 设置 CI/CD

本页介绍如何设置持续集成 (CI) 和持续部署 (CD) 管道。

## 概述 {#overview}

持续集成 (CI) 是频繁合并代码更改并在提交更改时自动测试更改的做法。持续部署 (CD) 意味着自动将集成更改部署到预发布或生产环境。两者结合，让你可以更频繁地改进对话机器人，并高效地测试和部署这些更改。

本指南将介绍特定于 Rasa 项目的 CI/CD 管道中应包含的内容。如何实施该管道取决于你。市面上有许多 CI/CD 工具，例如 [GitHub Actions](https://github.com/features/actions)、[GitLab CI/CD](https://docs.gitlab.com/ee/ci/)、[Jenkins](https://www.jenkins.io/doc/) 和 [CircleCI](https://circleci.com/docs/)。我们建议选择与你使用的任何 Git 存储库集成的工具。

## 示例 Rasa Pro CI/CD 管道 {#example-rasa-pro-cicd-pipeline}

Github Actions 工作流程示例如下所示：

=== "Rasa Pro >= 3.8.x"

    ```yaml
    name: Rasa Pro CI/CD Pipeline
    
    on:
      push:
        branches:
          - main
        paths:
          - 'data/**'
          - 'domain/**'
    
    jobs:
      train_test_deploy:
        runs-on: ubuntu-latest
    
        steps:
          - name: Checkout repository
            uses: actions/checkout@v3
    
          # Make sure to use the exact same Rasa version as in your TEST/PROD environment
          - name: Pull Rasa Pro image
            run: |
              docker pull europe-west3-docker.pkg.dev/rasa-releases/rasa-plus/rasa-plus:3.7.0-latest
    
          # Store the Rasa Pro license string as a secret RASA_PRO_LICENSE
          - name: Train Rasa model with Rasa Pro
            run: |
              docker run -v ${GITHUB_WORKSPACE}:/app \
                -e OPENAI_API_KEY=${OPENAI_API_KEY} \
                -e RASA_PRO_LICENSE=${RASA_PRO_LICENSE} \
                -e RASA_TELEMETRY_ENABLED=false \
                europe-west3-docker.pkg.dev/rasa-releases/rasa-plus/rasa-plus:3.7.0-latest \
                train --domain /app/domain --data /app/data --out /app/models
            env:
              OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
              RASA_PRO_LICENSE: ${{ secrets.RASA_PRO_LICENSE }}
    
          # Make sure to use your action server image and point rasa to the folder with your e2e tests (e.g. tests/e2e_test_cases.yml)
          # Use the same Rasa version as in the previous step
          # Your endpoints file should contain the action server url (e.g. http://action_server:5055/webhook)
          - name: Start Action Server and Test Rasa model with E2E tests
            run: |
              # Start the action server in the background
              docker run -d -p 5055:5055 --name action_server <YOUR-ACTION-SERVER-IMAGE>
    
              # Wait for the action server to become ready with a timeout
              echo "Waiting for action server to be ready..."
              timeout=60
              while ! curl --output /dev/null --silent --fail http://localhost:5055/health; do
                printf '.'
                sleep 5
                timeout=$((timeout-5))
                if [ "$timeout" -le 0 ]; then
                  echo "Action server did not become ready in time."
                  echo "Action server logs:"
                  docker logs action_server
                  exit 1
                fi
              done
              echo "Action server is ready."
    
              # Run E2E tests in the same step to ensure the action server is still running
              # Use container linking to allow communication
              docker run -v ${GITHUB_WORKSPACE}:/app \
                --link action_server:action_server \
                -e OPENAI_API_KEY=${OPENAI_API_KEY} \
                -e RASA_PRO_LICENSE=${RASA_PRO_LICENSE} \
                -e RASA_TELEMETRY_ENABLED=false \
                europe-west3-docker.pkg.dev/rasa-releases/rasa-plus/rasa-plus:3.7.0-latest \
                test e2e /app/tests/e2e_test_cases.yml --model /app/models --endpoints /app/endpoints.yml
            env:
              OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
              RASA_PRO_LICENSE: ${{ secrets.RASA_PRO_LICENSE }}
    
          # if tests fail, next steps won't be executed and model won't be uploaded to the bucket
          # Authenticate to Google Cloud again to avoid permission issues and use a service account with Storage Object Creator role
          - name: Authenticate to Google Cloud
            uses: google-github-actions/auth@v0.4.0
            with:
              credentials_json: '${{ secrets.GCP_SA_KEY }}'
          - name: Configure Google Cloud project
            run: |
              echo $GCP_SA_KEY | gcloud auth activate-service-account --key-file=-
            env:
              GCP_SA_KEY: ${{ secrets.GCP_SA_KEY }}
          # Upload the trained model to the Model Storage bucket associated with     your deployment
          - name: Upload model to Google Cloud Storage
            run: |
              gsutil cp -r models/* gs://my-model-storage/
    ```

=== "Rasa Pro <=3.7.x"

    ```yaml
    name: Rasa Pro CI/CD Pipeline
    
    on:
      push:
        branches:
          - main
        paths:
          - 'data/**'
          - 'domain/**'
    
    jobs:
      train_test_deploy:
        runs-on: ubuntu-latest
    
        steps:
          - name: Checkout repository
            uses: actions/checkout@v3
    
          # Make sure to use the exact same Rasa version as in your TEST/PROD environment
          - name: Pull Rasa Pro image
            run: |
              docker pull europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro:3.8.0-latest
    
          # Store the Rasa Pro license string as a secret RASA_PRO_LICENSE
          - name: Train Rasa model with Rasa Pro
            run: |
              docker run -v ${GITHUB_WORKSPACE}:/app \
                -e OPENAI_API_KEY=${OPENAI_API_KEY} \
                -e RASA_PRO_LICENSE=${RASA_PRO_LICENSE} \
                -e RASA_TELEMETRY_ENABLED=false \
                europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro:3.8.0-latest \
                train --domain /app/domain --data /app/data --out /app/models
            env:
              OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
              RASA_PRO_LICENSE: ${{ secrets.RASA_PRO_LICENSE }}
    
          # Make sure to use your action server image and point rasa to the folder with your e2e tests (e.g. tests/e2e_test_cases.yml)
          # Use the same Rasa version as in the previous step
          # Your endpoints file should contain the action server url (e.g. http://action_server:5055/webhook)
          - name: Start Action Server and Test Rasa model with E2E tests
            run: |
              # Start the action server in the background
              docker run -d -p 5055:5055 --name action_server <YOUR-ACTION-SERVER-IMAGE>
    
              # Wait for the action server to become ready with a timeout
              echo "Waiting for action server to be ready..."
              timeout=60
              while ! curl --output /dev/null --silent --fail http://localhost:5055/health; do
                printf '.'
                sleep 5
                timeout=$((timeout-5))
                if [ "$timeout" -le 0 ]; then
                  echo "Action server did not become ready in time."
                  echo "Action server logs:"
                  docker logs action_server
                  exit 1
                fi
              done
              echo "Action server is ready."
    
              # Run E2E tests in the same step to ensure the action server is still running
              # Use container linking to allow communication
              docker run -v ${GITHUB_WORKSPACE}:/app \
                --link action_server:action_server \
                -e OPENAI_API_KEY=${OPENAI_API_KEY} \
                -e RASA_PRO_LICENSE=${RASA_PRO_LICENSE} \
                -e RASA_TELEMETRY_ENABLED=false \
                europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro:3.8.0-latest \
                test e2e /app/tests/e2e_test_cases.yml --model /app/models --endpoints /app/endpoints.yml
            env:
              OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
              RASA_PRO_LICENSE: ${{ secrets.RASA_PRO_LICENSE }}
    
          # if tests fail, next steps won't be executed and model won't be uploaded to the bucket
          # Authenticate to Google Cloud again to avoid permission issues and use a service account with Storage Object Creator role
          - name: Authenticate to Google Cloud
            uses: google-github-actions/auth@v0.4.0
            with:
              credentials_json: '${{ secrets.GCP_SA_KEY }}'
          - name: Configure Google Cloud project
            run: |
              echo $GCP_SA_KEY | gcloud auth activate-service-account --key-file=-
            env:
              GCP_SA_KEY: ${{ secrets.GCP_SA_KEY }}
          # Upload the trained model to the Model Storage bucket associated with your deployment
          - name: Upload model to Google Cloud Storage
            run: |
              gsutil cp -r models/* gs://my-model-storage/
    ```
