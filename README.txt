Projeto Criado com o Auxílio do professor Daniel faz parte do programa
Ligthouse da Indicium. 

Descrição do passo a passo utilizado:

1) Donwloads necessários: 
Primeiro, como o sistema operacional utilizado foi windows, é necessário baixar a linguagem python para a máquina no seguinte link: https://www.python.org/ 
Realizado o donwload do python é ideal utilizar alguma IDE para poder utilizado os códigos e comandos  python. A IDE utilizada foi a Visual Studio Code (Vscode), encontrado no seguinte link: https://code.visualstudio.com/

2) Criação do Ambiente de Trabalho:
Para criação de um projeto em python é recomendável utilizar ambientes virtuais, pois, um ambiente virtual empacota todas as dependências que um projeto precisa e armazena em um diretório, fazendo com que nenhum pacote seja instalado diretamente no sistema operacional. Sendo assim, cada projeto pode possuir seu próprio ambiente e, consequentemente, suas bibliotecas em versões específicas. Para criar o ambiente virtual abre-se o vscode e coloca-se o seguinte comando:

cd meuprojeto/
python -m venv venv
source venv/bin/activate

Para Windows, o comando deverá ser o seguinte>

PS> python -m venv venv
PS> venv\Scripts\activate
(venv) PS>

3) Criação do Verificador de Sites:
Inicialmente é criado uma simples função em Python que que verifica se um site está online usando o pacote urllib e http. Para isso coloca-se o código abaixo em uma sessão interativa:

>>> from http.client import HTTPConnection

>>> connection = HTTPSConnection("indicium.tech", port=80, timeout=10)
>>> connection.request("HEAD", "/")

>>> response = connection.getresponse()
>>> response.getheaders()
[('Date', 'Tue, 20 Sep 2022 18:10:37 GMT'), ('Content-Type', 'text/html'), ('Content-Length', '178'), ('Connection', 'keep-alive'), ('Cache-Control', 'max-age=600'), ('Location', 'https://www.globo.com/')]

Esse código inicia criando uma conexão para uma URL usando a porta padrão HTTP 80. Em seguida, fazemos uma requisição para o caminho padrão "/" usando o método .request(). A resposta dessa requisição é obtida pelo método .getresponse(). Finalmente, para evitar trazer todo o arquivo do site trazemos somente o HEADER. Se o request tiver sucesso, o site está online. Caso contrário, retorná um erro.

Esta lógica é incluida em um arquivo checker.py que vai lidar com possíveis erros:

# checker.py
from http.client import HTTPConnection
from urllib.parse import urlparse

def site_is_online(url, timeout=2):
    """Return True if the target URL is online.

    Raise an exception otherwise.
    """
    # Defines a generic Exception as placeholder
    error = Exception("Ops, algo errado.")
    # Parses URL and finds host
    parser = urlparse(url)
    host = parser.netloc or parser.path.split("/")[0]
    # Starts a for loop using HTTP and HTTPs ports
    for port in (80, 443):
        connection = HTTPConnection(host=host, port=port, timeout=timeout)
        try:
            connection.request("HEAD", "/")
            return True
        except Exception as e:
            error = e
        finally:
            connection.close()
    raise error

Para testar a função recém criada, abra uma sessão interativa no Python e rode o código abaixo:

>>> from sitechecker.checker import site_is_online

>>> site_is_online("indicium.tech")
True

>>> site_is_online("incidium.tech")

4) Criação do CLI:
Para criar uma aplicação de linha de comando (CLI) é utilizado o argparse do python. Para isso, precisa-se criar uma classe ArgumentParser que recebe argumentos da linha de comando.

# cli.py

import argparse

def read_user_cli_args():
    """Handle the CLI arguments and options."""
    parser = argparse.ArgumentParser(
        prog="sitechecker", description="Teste a disponibilidade de uma URL"
    )
    parser.add_argument(
        "-u",
        "--urls",
        metavar="URLs",
        nargs="+",
        type=str,
        default=[],
        help="insira um ou mais URLs",
    )
    return parser.parse_args()

Assim, têm-se uma função que recebe argumentos do CLI, porém, é necessário retornar algo para nosso usuário. Para isso, é criada uma função que retorna uma mensagem de sucesso se o site estiver online ou de fracasso se o site não está online.

# cli.py
# ...

def display_check_result(result, url, error=""):
    """Display the result of a connectivity check."""
    print(f'O status da "{url}" é:', end=" ")
    if result:
        print('"Online!" 👍')
    else:
        print(f'"Offline?" 👎 \n  Erro: "{error}"')

5) Unindo todo o código:
Após realizar toda a lógica separada, é o momento de uni-lá no arquivo checker.py com cli.py. Para isso é criado um arquivo __main__.py que permite executar o pacote como um executável usando python -m <nome_do_pacote>.

Então em primeiro lugar uma função main(), que lê os argumentos do CLI, é criada e chamará as demais funções necessárias:

# __main__.py

import sys

from sitechecker.cli import read_user_cli_args

def main():
    """Run Site Checker."""
    user_args = read_user_cli_args()
    urls = user_args.urls
    if not urls:
        print("Erro: sem URLs para analisar.", file=sys.stderr)
        sys.exit(1)
    _site_check(urls)

Logo vemos que esse código não irá rodar porque ainda não está definida a função _site_check(). Esta função vai iterar sobre uma lista de URLs obtida dos argumentos do CLI e aplicar a função site_is_online() que foi definida anteriormente. Em caso de sucesso, ela retornará True. E False se a consulta retornar um erro.

# __main__.py

import pathlib
import sys

from sitechecker.cli import read_user_cli_args

def main():
    # ...

def _site_check(urls):
    for url in urls:
        error = ""
        try:
            result = site_is_online(url)
        except Exception as e:
            result = False
            error = str(e)
        display_check_result(result, url, error)
        

