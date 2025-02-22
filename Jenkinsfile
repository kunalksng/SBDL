pipeline {
    agent any

    environment {
        SPARK_HOME = "/usr/local/Cellar/apache-spark/3.5.4/libexec"
        PATH = "$SPARK_HOME/bin:$SPARK_HOME/sbin:$PATH"
        REPO_URL = 'git@github.com:kunalksng/SBDL.git'
        BRANCH_NAME = 'release' // Change to 'master' for production deployment
        PYTHONPATH = "$WORKSPACE" // Add the workspace directory to PYTHONPATH
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH_NAME}", url: "${REPO_URL}"
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                echo "Setting up Python environment..."
                python3 -m venv venv          # Create a virtual environment
                . venv/bin/activate            # Activate the virtual environment
                pip install --upgrade pip       # Upgrade pip to the latest version
                pip install pipenv             # Install pipenv within the virtual environment
                pipenv --python python3 sync  # Sync dependencies with pipenv
                echo "Python environment setup complete."
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                echo "Running tests..."
                . venv/bin/activate
                export PYTHONPATH=$PYTHONPATH:$WORKSPACE  # Ensure lib is in PYTHONPATH
                pipenv run pytest
                echo "Tests completed."
                '''
            }
        }

        stage('Package Application') {
            when {
                anyOf { branch 'master'; branch 'release' }
            }
            steps {
                sh '''
                echo "Packaging application..."
                zip -r sbdl.zip lib conf log4j.properties main.py sbdl_submit.sh
                echo "Application packaged successfully."
                '''
            }
        }

        stage('Deploy') {
            when {
                branch 'release'  // Change to 'master' for production deployment
            }
            steps {
                sh '''
                echo "Deploying application..."
                mkdir -p /Users/kunal/deployments/sbdl
                cp sbdl.zip /Users/kunal/deployments/sbdl/
                cp log4j.properties /Users/kunal/deployments/sbdl/
                cp main.py /Users/kunal/deployments/sbdl/
                cp sbdl_submit.sh /Users/kunal/deployments/sbdl/
                echo "Deployment Complete!"
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!
