name: CI/CD for Python Service

on:
  push:
    branches:
      - main  # Change this to your default branch

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.x'

    - name: Install dependencies
      run: |
        pip install -r requirements.txt

    - name: Lint code
      run: |
        pip install flake8
        flake8 .

    - name: Run tests
      run: |
        pip install pytest
        pytest

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Install SSH key
      uses: webfactory/ssh-agent@v0.7.0
      with:
        ssh-private-key: ${{ secrets.EC2_KEY }}

    - name: Copy files to EC2 instance
      run: |
        scp -o StrictHostKeyChecking=no -r ./ ec2-user@${{ secrets.EC2_HOST }}:/home/${{ secrets.EC2_USER }}/your-app-directory

    - name: SSH into EC2 and restart service
      run: |
        ssh -o StrictHostKeyChecking=no ec2-user@${{ secrets.EC2_HOST }} << 'EOF'
        cd /home/${{ secrets.EC2_USER }}/your-app-directory
        # Install dependencies
        pip3 install -r requirements.txt
        # Restart the application (example for Flask/Django)
        sudo systemctl restart your-app.service
        EOF
