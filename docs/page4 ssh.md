# ssh и Yandex-cloud


# 1. Создание SSH

```bash
# Создаём ssh ключ
tiv@DESKTOP-0A8EL6K:~$ ssh-keygen
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/tiv/.ssh/id_ed25519):
Created directory '/home/tiv/.ssh'.

# Кодовое слово можно пропустить Enter
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/tiv/.ssh/id_ed25519
Your public key has been saved in /home/tiv/.ssh/id_ed25519.pub

The key fingerprint is:
SHA256:3eohroXeToz24HtTmZZtQxMAZmMDzX1U8zaziyf1cPE tiv@DESKTOP-0A8EL6K

The keys randomart image is:
+--[ED25519 256]--+
|      .+Bo.o..o  |
|       +oo. o  o |
|           . . +o|
|         . .o  .*|
|        S .*...oE|
|       +  *.+ ooo|
|      = =oo. + o.|
|     + Boo .  o  |
|      ===..      |
+----[SHA256]-----+

# Переходим в папку с созданным ключом и проверяем их наличие:
tiv@DESKTOP-0A8EL6K:~$ cd /home/tiv/.ssh
tiv@DESKTOP-0A8EL6K:~/.ssh$ ll

total 16
drwx------ 2 tiv tiv 4096 Jul 29 15:34 ./
drwxr-x--- 9 tiv tiv 4096 Jul 29 15:34 ../
-rw------- 1 tiv tiv  411 Jul 29 15:34 id_ed25519
-rw-r--r-- 1 tiv tiv  101 Jul 29 15:34 id_ed25519.pub

# Открываем публичный ключ, копируем его и вставляем в яндекс-клауд
tiv@DESKTOP-0A8EL6K:~/.ssh$ cat id_ed25519.pub

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAkwA1mR1zUsLPR8EQrp62R+tp/O7asUE9Y3njbQcOId 
tiv@DESKTOP-0A8EL6K

# Подключаемся к виртуальной машине. Эту команду берём из яндекс-клауд:
tiv@DESKTOP-0A8EL6K:~/.ssh$ ssh -l tiv-vm 158.160.148.237

# Проверяем имя пользователя виртуальной машины
tiv-vm@compute-vm-2-2-20-ssd-1753792566694:~$ whoami
tiv-vm
```



## Загрузка файлов через ssh в облачную ВМ

Если хотим на ВМ выложить файл с помощью ssh-ключа, созданного ранее:
1) Открываем терминал на локальной машине
2) Переходим в директорию с нужным файлом
3) Используем команду

`scp deploy.yaml tiv-vm@158.160.140.141:~/jopa/popa`

где

- `deploy.yaml` - файл
- `tiv-vm` - название виртуальной машины
- `158.160.140.141` - IP виртуальной машины
- `~/jopa/popa` - путь до нужной папки на облаке


---

# 2. Kubernetes context, configs

Создаём кластер в Яндекс-клауд:
- Задаём характеристики кластера и мастер-ноды	
- Создаём в другой вкладке рабочие ноды
- При создании указываем ранее созданный ключ (выпадет из списка)
- Вносим изменения в файл **kubeconfig** командой, которую подскажет Яндекс-клауд. 
 
Например:
`yc managed-kubernetes cluster get-credentials --id catgrgutqjr7iekuqn2r --external`

Кластер поднимется в течении 10 минут.

Поскольку это не единственный наш кластер, то нужно понимать которым из них мы в данный момент управляем.

Ключевой файл **kubeconfig** содержит всю информацию о кластерах. Располагается на пути:
`~/.kube/config` ^0ea0ef

%% можно указать другой файл с помощью переменной окружения KUBECONFIG или флага --kubeconfig %%

Следующие команды обращаются непосредственно к нему.

```bash
# выводит настройки файла kubeconfig в формате YAML. Она показывает информацию о кластерах, контекстах и пользователях, определённых в файле конфигурации.

kubectl config view
```

```bash
# выводит таблицу со всеми контекстами в файле kubeconfig. Она показывает информацию о текущем контексте, названии кластера, данных авторизации и пространстве имён. * - текущий контекст
kubectl config get-contexts

# Похожая команда, выводит только текущий контекст:
kubectl config current-context
```


```bash
# переключиться на управлением кластера с именем <name>:
kubectl config use-context <name>
```
