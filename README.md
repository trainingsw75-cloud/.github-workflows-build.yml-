name: Build Android APK

on: 
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  build-apk:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Python 3.8
      uses: actions/setup-python@v4
      with:
        python-version: '3.8'

    - name: Install system dependencies
      run: |
        sudo apt-get update
        sudo apt-get install -y \
          python3-pip \
          openjdk-11-jdk \
          zip \
          unzip \
          git \
          autoconf \
          libtool \
          pkg-config \
          zlib1g-dev \
          libncurses5-dev \
          libncursesw5-dev \
          cmake \
          ninja-build \
          libffi-dev \
          libssl-dev

    - name: Upgrade system tools
      run: |
        python -m pip install --upgrade pip wheel setuptools virtualenv

    - name: Install Buildozer and dependencies
      run: |
        pip install --user buildozer cython
        echo "$HOME/.local/bin" >> $GITHUB_PATH

    - name: Debug - Show environment
      run: |
        which buildozer || echo "buildozer not in PATH"
        buildozer --version || echo "Cannot run buildozer"
        python --version
        pip --version

    - name: Initialize Buildozer if spec doesn't exist
      run: |
        if [ ! -f "buildozer.spec" ]; then
          buildozer init
          echo "Created new buildozer.spec"
        else
          echo "buildozer.spec already exists"
        fi

    - name: Show buildozer.spec content
      run: |
        cat buildozer.spec || echo "No buildozer.spec file"

    - name: Build APK with Buildozer
      run: |
        buildozer -v android debug
      
    - name: Check if APK was created
      run: |
        if ls bin/*.apk 1> /dev/null 2>&1; then
          echo "🎉 APK created successfully!"
          ls -la bin/
        else
          echo "❌ No APK found in bin directory"
          echo "Contents of current directory:"
          ls -la
          echo "Contents of bin directory (if exists):"
          ls -la bin/ || echo "No bin directory"
        fi

    - name: Upload APK artifact
      if: success()
      uses: actions/upload-artifact@v4
      with:
        name: android-apk
        path: bin/*.apk
        retention-days: 3
