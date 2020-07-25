# python 개발 환경 다시 만들기



## 1. anaconda 완전히 삭제

>  공식 홈페이지 Uninstall 링크 : https://docs.anaconda.com/anaconda/install/uninstall/

1. `conda install anaconda-clean` 로 `anaconda-clean` 설치
2. 설치 후 `anaconda-clean --yes` 실행
3. `~/bash-profile` 혹은 `~/zshrc` 에서 `export PATH="/Users/{user_name}/anaconda3/bin:$PATH"` 삭제 후 저장
4. `rm -rf ~/anaconda3` 실행
5. `rm -rf ~/.anaconda_backup/` 실행



## 2. Pyenv 및 python 설치

>  참고 링크 : https://blog.jayway.com/2019/12/28/pyenv-poetry-saviours-in-the-python-chaos/

터미널 열고 아래 명령어 하나씩 입력하면 된다.

```bash
# Install pyenv.
brew install pyenv

# Add pyenv initializer to shell startup script.
echo -e '\nif command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi' >> ~/.bash_profile

# Reload your profile.
source ~/.bash_profile
```

여기까지 완료가 되었으면, 다음 명령어로 install 가능한 목록을 볼 수 있다.

```bash
pyenv install --list
```

다음과 같이 특정 파이썬 버전을 설치할 수 있다.

```python
pyenv install  3.7.8
```

다음과 같이 현재 설치된 파이썬 버전을 볼 수 있다.

```bash
pyenv versions
```

다음 명령어로 global 파이썬 인터프리터를 설정할 수 있다.

```bash
pyenv global 3.7.8
```

다음 명령어로 현재 사용 중인 파이썬 버전을 볼 수 있다.

```bash
pyenv version
```



## 3. Poetry 설치

> 참고 링크 :
>
> -  https://blog.jayway.com/2019/12/28/pyenv-poetry-saviours-in-the-python-chaos/
> - https://spoqa.github.io/2019/08/09/brand-new-python-dependency-manager-poetry.html
> - https://lhy.kr/python-poetry



터미널 열고 아래 명령어 하나씩 입력하면 된다.

``` bash
# Install poetry via curl
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python

# Add poetry to your shell path
echo -e 'export PATH="$HOME/.poetry/bin:$PATH"' >> ~/.bash_profile
# zsh 사용하는 사람은 >> ~/.zshrc 로 하면 됨.

# Configure poetry to create virtual environments inside the project's root directory
poetry config virtualenvs.in-project true
```

`poetry completion` 은 shell 에서 `poetry` 를 입력하고 tab 을 누르면 명령어 목록이 뜨는 기능이다.
이를 사용하려면 `~/.zshrc` 을 열어 plugin 에 다음가 같이 추가해줘야 한다.

```python
# ~/zshrc
...
plugins=(
    poetry
    ...
)
...
```

다음 명령어로 poetry 를 시작할 수 있다.

```bash
poetry init
```

다음 명령어로 패키지를 추가 / 삭제 할 수 있다.

```
poetry add <패키지명>
poetry remove <패키지명>
```

다음 명령어로 설치된 패키지를 확인할 수 있다.

```
poetry show --tree
poetry show --no-dev --tree
```

다음 명령어로 `requirements.txt` 를 만들 수 있다.

```
poetry export -f requirements.txt > requirements.txt
```

