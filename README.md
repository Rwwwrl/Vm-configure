# Гайд по настройке виртуалки

1. Установить **openssh-server** (без него не пройдет подключение по _ssh_):

```bash
sudo apt install net-tools
sudo apt install openssh-server
```

2. Установка всех **dependencies**

```bash
yes | sudo apt update
yes | sudo apt upgrade
yes | sudo apt install build-essential libssl-dev libpq-dev python3-venv python3-pip python3-wheel
yes | sudo apt install curl  # optional
yes | sudo apt install mlocate  # optional
yes | sudo apt install fzf  # optional

pip3 install wheel  # в venv
```

3. Установка и настройка **git**

```bash
yes | sudo apt install git
ssh-keygen
cat /home/<UserName>/.ssh/id_rsa.pub
```

4. **[OPTIONAL]** Установка и настройка **vim**

```bash
yes | sudo apt install vim
```

[Настройка](https://github.com/Rwwwrl/minimal-vim-config)

5. Установка и настройка **zsh**

```bash
sudo apt install zsh # тут выбираем 0
chsh -s $(which zsh)
```

```bash
sudo reboot
```

```bash
yes | sudo apt-get install fonts-powerline
# установка ohMyZsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

```

```bash
mkdir ~/code_folder  # как основная папка где будет находиться проект
mkdir -p ~/.config/systemd/user  # чтобы располагать наши сервисы systemd тут
```

```bash
vim ~/.zshrc
```

```bash

# чтобы PATH видел питоновские пакеты
export PATH=$HOME/.local/bin:$PATH

# для активации autosuggestions
plugins=(
    # other plugins...
    zsh-autosuggestions
)

# добавить в конец .zshrc
export TERM='ms-terminal' # если сидишь под WindowsTerminal

export NGINX='/etc/nginx/'
export SERVICES="$HOME/.config/systemd/user" # теперь к ней можно обращаться как "cd $SERVICES"

export CODE_FOLDER="$HOME/code_folder/"

export EDITOR='/usr/bin/vim'  # нужно чтобы sudoedit врубал именно vim, а не nano

alias code_folder="cd $CODE_FOLDER"
alias python='python3'
alias pip='pip3'
alias aenv="source env/bin/activate"
alias sudoe="sudoedit"
alias sudo="sudo " # чтобы работали алиасы в sudo моде
alias ctl='systemctl --user'
alias sctl='sudo systemctl'
```

```python
source ~/.zshrc
```

6. **[OPTIONAL]** Установка и настройка **tmux**:

```bash
yes | sudo apt install tmux
```

создать и скопировать в файл _.tmux.conf_

```bash
set -g base-index 1

# Automatically set window title
set-window-option -g automatic-rename on
set-option -g set-titles on

bind-key v split-window -h -c "#{pane_current_path}"
bind-key s split-window -v -c "#{pane_current_path}"

bind-key -r J resize-pane -D 5
bind-key -r K resize-pane -U 5
bind-key -r H resize-pane -L 5
bind-key -r L resize-pane -R 5

# Vim style pane selection
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Use Alt-vim keys without prefix key to switch panes
bind -n M-h select-pane -L
bind -n M-j select-pane -D
bind -n M-k select-pane -U
bind -n M-l select-pane -R

# Shift arrow to switch windows
bind -n S-Left  previous-window
bind -n S-Right next-window

# Reload tmux config
bind r source-file ~/.tmux.conf \; display "Reloaded!"
```

Или просто:

```bash
printf 'set -g base-index 1\n# Automatically set window title\nset-window-option -g automatic-rename on\nset-option -g set-titles on\nbind-key v split-window -h -c "#{pane_current_path}"\nbind-key s split-window -v -c "#{pane_current_path}"\nbind-key -r J resize-pane -D 5\nbind-key -r K resize-pane -U 5\nbind-key -r H resize-pane -L 5\nbind-key -r L resize-pane -R 5\n# Vim style pane selection\nbind h select-pane -L\nbind j select-pane -D\nbind k select-pane -U\nbind l select-pane -R\n# Use Alt-vim keys without prefix key to switch panes\nbind -n M-h select-pane -L\nbind -n M-j select-pane -D\nbind -n M-k select-pane -U\nbind -n M-l select-pane -R\n# Shift arrow to switch windows\nbind -n S-Left  previous-window\nbind -n S-Right next-window\n# Reload tmux config\nbind r source-file ~/.tmux.conf \; display "Reloaded!"' > ~/.tmux.conf
```

```bash
tmux source-file ~/.tmux.conf
```

7. Установка **postgres**

```bash
yes | sudo apt install postgresql postgresql-contrib

sudoe /etc/postgresql/12/main/pg_hba.conf # чтобы изменить на trust

/etc/postgresql/12/main/ # conf directory

sudo pg_ctlcluster 12 main start
```

8. Установка **redis**

```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

yes | sudo apt-get install redis
```

9. Установка **gunicorn**
   Находясь в _venv_:

```bash
pip install gunicorn
```

10. Установка **nginx**:

```bash
yes | sudo apt install nginx
```

### DEPLOY

Я ориентировался на [эту](https://habr.com/ru/post/546778/) статью и разворачивал [этот](https://github.com/Rwwwrl/simple_django_project_for_deploy) проект

#### Django

Добавляем **MEDIA_ROOT** и **STATIC_ROOT**

```python
...
DEBUG = False
...
STATIC_URL = '/static/'  # без этого не получится выполнить collectstatic
STATIC_ROOT = BASE_DIR / 'static'
...
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
...
```

Соответственно создаем эти папки

```bash
mkdir static media
```

Перемещаем всю статику в проекте в папку _STATIC_ROOT_

```python
python manage.py collectstatic
```

#### Gunicorn

по пути _$SERVICES_ создаем два файла **_simple_django_gunicorn.service_** и **_simple_django_gunicorn.socket_**

**simple_django_gunicorn.service:**

```bash
vim $SERVICES/simple_django_gunicorn.service
```

```bash
[Unit]
Description=gunicorn daemon for simple_django_app
Requires=simple_django_gunicorn.socket
After=network.target redis.service postgresql.service

[Service]
# абсолютный путь до файла manage.py
WorkingDirectory=/home/<UserName>/code_folder/simple_app/simple_django_project_for_deploy

# абсолютный путь до gunicorn в venv, (полный путь до сокета писать не нужно)
ExecStart=/home/<UserName>/code_folder/simple_app/env/bin/gunicorn --workers 3 --bind unix:simple_app.sock main.wsgi

[Install]
WantedBy=multi-user.target
```

> **Нe забываем** поменять \<UserName>

**simple_django_gunicorn.socket:**

```bash
vim $SERVICES/simple_django_gunicorn.socket
```

```bash
[Unit]
Description=gunicorn socket for simple_django_app

[Socket]
# путь до места где будет находиться сокет
# ( т.к. наши сервисы мы запускаем не от sudo, то и использовать /run/gunicorn.sock мы не можем)
ListenStream=/home/<UserName>/code_folder/simple_app/simple_django_project_for_deploy/main/simple_app.sock

[Install]
WantedBy=sockets.target
```

> **Не забываем** поменять \<UserName>

> Отмечу, что мы используем socket нежели TCP протокол

```bash
ctl daemon-reload
```

Теперь мы можем воспользоваться следующими командами:

```bash
ctl enable simple_django_gunicorn.service # чтобы сервис всегда включался автоматически при заходе в систему
ctl start simple_django_gunicorn.service # включить сервис здесь и сейчас

ctl stop simple_django_gunicorn.service # остановить работу сервиса здесь и сейчас
ctl disable simple_django_gunicorn.service # отключить автозапуск сервиса при входе в систему

```

#### Nginx

Создаем файл _django_simple_app_ в _/etc/nginx/sites-available/_

```bash
sudoe $NGINX/sites-available/django_simple_app
```

```bash
server {
    listen 80;
    server_name django_simple_app;

    location = /favicon.ico { access_log off; log_not_found off;  }
    location /static/ {
            root /home/<UserName>/code_folder/simple_app/simple_django_project_for_deploy;           #путь до static каталога (не включая саму папку static)

    }

    location /media/ {
            root /home/<UserName>/code_folder/simple_app/simple_django_project_for_deploy;           #путь до media каталога (не включая саму папку media)

    }

    location / {
            include proxy_params;
            proxy_pass http://unix:/home/<UserName>/code_folder/simple_app/simple_django_project_for_deploy/main/simple_app.sock;  # путь до нашего сокета

    }

}
```

> **Не забываем** поменять \<UserName>
> **Не забываем** удалить default файл
>
> ```bash
> sudo rm sites-enabled/default
> ```

Делаем ссылку на этот файл в директорию _/etc/nginx/sites-enabled_

```bash
sudo ln -s $NGINX/sites-available/django_simple_app $NGINX/sites-enabled
```

```bash
sudo nginx -t  # проверяем тесты

sudo service nginx start  # стартуем наш сервер
```

#### TODO:

- **SSL** сертификат
