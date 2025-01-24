node {
    def TARGET_SERVER = "chatapp@10.0.3.11"
    def APP_DIR = "/new_chatapp"
    def APP_SUBDIR = "fundoo"
    def VENV_DIR = "$APP_DIR/venv"

    try {
        stage('Checkout Code') {
            echo "Workspace location: $WORKSPACE"
            echo "Checking out the code from the repository..."
            git url: 'https://github.com/Aayush99Sharma/chatapp-modified-ubuntu-22.04.git', branch: 'main'
            echo "Code checked out in workspace at: $WORKSPACE"
        }

        stage('Sync Code') {
            echo "Starting CI/CD pipeline for backend server..."
            echo "Syncing application code from $WORKSPACE to the backend server ($TARGET_SERVER)..."
            sh """
            rsync -avz --exclude '.git' $WORKSPACE/ $TARGET_SERVER:$APP_DIR
            """
        }

        stage('Build') {
            echo "Building the application on the backend server..."
            def buildStatus = sh(returnStatus: true, script: """
            ssh $TARGET_SERVER 'set -e
            echo "Navigating to application directory..."
            cd $APP_DIR

            echo "Activating the virtual environment..."
            source $VENV_DIR/bin/activate

            echo "Installing dependencies from requirements.txt..."
            pip install -r requirements.txt'
            """)
            if (buildStatus != 0) {
                error "Build stage failed with exit code: ${buildStatus}"
            }
        }

        stage('Deploy') {
            echo "Deploying the application on the backend server..."
            def deployStatus = sh(returnStatus: true, script: """
            ssh $TARGET_SERVER 'set -e
            echo "Navigating to the subdirectory containing manage.py..."
            cd $APP_DIR/$APP_SUBDIR

            echo "Running Django database migrations using manage.py..."
            python3 manage.py makemigrations

            echo "Running Django database migrations using the run_migrate.sh script..."
            bash ~/run_migrate.sh

            echo "Restarting the Django service..."
            sudo systemctl restart django.service

            echo "Checking the status of the Django service..."
            sudo systemctl status django.service'
            """)
            if (deployStatus != 0) {
                error "Deploy stage failed with exit code: ${deployStatus}"
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        echo "Error: ${e.getMessage()}"
        throw e
    } finally {
        echo "Final build result: ${currentBuild.result}"
        if (currentBuild.result == 'SUCCESS') {
            echo "CI/CD pipeline completed successfully!"
        } else {
            echo "CI/CD pipeline failed. Please check the logs for errors."
        }
    }
}
