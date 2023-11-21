---
title: Fuzz模糊测试简介
date: 2023-01-01 21:10:06
categories:
- 学习
tags: 
- fuzz
- 模糊测试
- pwn
---
credit: @SecureNexusLab
## 模糊测试
模糊测试(fuzz testing, fuzzing)是一种软件测试技术。  
其核心思想是将自动或半自动生成的随机数据输入到一个程序中，并监视程序异常，如崩溃，断言(assertion)失败，以发现可能的程序错误，比如内存泄漏。模糊测试常常用于检测软件或计算机系统的安全漏洞。
## 模糊测试通用流程
对于一个模糊测试通用的测试流程，其描述如下:
1. 首先被测系统要动态运行，被测工具要与被测系统建立联系;
2. 测试工具根据变异算法对原始测试用例进行变异;
3. 测试工具将生成的测试用用例发送给被测系统的入;
4. 测试工具监控被测系统的运行状态（是否可以正常响应，是否可以正常运行）;
5. 测试工具如果发现缺陷，进行记录;
6. 重复上述操作，直至测试结束。


## 简单示例
```c
#include <stdio.h>
#include <string.h>
void vulnerable_function(char *input)
{
    char buffer[100];
    strcpy(buffer, input);
    printf("Received: %s\n", buffer);
}
int main()
{
    char input[200];
    printf("Enter data: ");
    gets(input);
    vulnerable_function(input);
    return 0;
}
```
当数据长度超过100时，目标程序会发生缓冲区溢出，可能导致程序崩溃或其他未预期的行为。 

对于测试脚本
```python
import subprocess
import random
import string

def random_string(length):
    return ''.join(random.choice(string.ascii_letters) for i inrange(length))

#Fuzzing loop
for _ in range(1000):
    data = random_string(random.randint(1,200))
    process = subprocess.Popen(['./vulnerable_program'],stdin=subprocess.PIPE)
    process.communicate(data.encode())
```
通过不断录入随机语句，触发栈溢出。
## 模糊测试应用
- 发现后门。通过不断输入随机语句可以触发包含后门的某些片段。  
    假设我们有一个程序，该程序在用户输入特定的字符串时，会触发隐藏的后门功能
    ```python
    def execute_command(command):# Hidden backdoor
        if command == "open_backdoor":# 隐藏指令
            return "Backdoor opened!"
        elif command =="normal_operation":
            return "Executing normal operation..."
        else:
            return "Unknown command!"
        
    # Test
    cmd = input("Enter command: ")
    print(execute_command(cmd))
    ```
- 破解秘钥硬编码。  
    假设我们有一个简单的程序，该程序使用硬编码的密钥来验证用户输入的密码
    ```python
    def check_password(input_password):
        # Hardcoded secret key
        secret_key = "s3cr3t_k3y"
        if input _password == secret_key:
            return "Access granted!"
        else:
            return "Access denied!"
    # Test
    password = input("Enter password: ")
    print(check_password(password))
    ```


