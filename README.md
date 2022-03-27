# Гайд по настройке WSL

1. Установка всех dependencies

```bash
sudo apt update
sudo apt upgrade
sudo apt install build-essential libssl-dev
sudo apt install curl
```

2. Установка **git**

```bash
sudo apt install git
```

3. Изменение **PS1** в **.bashrc**

```bash
# добавить в конец .bashrc
PS1="\[\e[34m\](\u) \[\e[31m\] \w \[\e[0m\] \$ "
```

2. Установка и настройка **vim**

```bash
sudo apt install vim
```

5. Чтобы можно было работать с ***systemctl***:
[ссылка](https://gist.github.com/djfdyuruiry/6720faa3f9fc59bfdf6284ee1f41f950)