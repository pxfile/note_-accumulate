# MAC 运行gradle报permission denied错误

命令行中输入：./gradlew xxxx  

出现下错误：

**zsh: permission denied: ./gradlew**

解决方式：

先输入：**chmod +x gradlew**，再输入你的命令行./gradlew xxxx 即可