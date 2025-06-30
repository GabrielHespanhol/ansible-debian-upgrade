# Atualização de Versão do Debian com Ansible: Debian 10 -> Debian 11 -> Debian 12
Este projeto Ansible tem como objetivo automatizar o processo de atualização de sistemas Debian, passando de forma segura do Debian 10 (Buster) para o Debian 11 (Bullseye) e, subsequentemente, para o Debian 12 (Bookworm).

**Atenção** : A execução de atualizações de sistema operacional é uma operação crítica. Leia este README completamente e entenda os riscos antes de prosseguir. Faça backups!

> Manter sistemas operacionais atualizados é fundamental para a segurança, estabilidade e acesso a novos recursos. Este playbook Ansible facilita o processo de upgrade de versões do Debian, minimizando a intervenção manual e garantindo um processo mais consistente em múltiplos servidores.

## Estrutura do Projeto
A organização deste projeto Ansible segue algumas boas práticas, facilitando a compreensão e manutenção.

```yaml
.
├── ansible.cfg              # Configurações globais do Ansible
├── inventory                # Definição dos hosts alvo
│   └── hosts
├── playbooks                # Playbooks principais
│   └── update_debian.yml
├── roles                    # Roles Ansible para tarefas específicas
│   ├── debian_10_to_11      # Role para atualizar do Debian 10 para o Debian 11
│   │   ├── files            # Arquivos estáticos (ex: sources.list)
│   │   │   └── sources.list
│   │   ├── handlers         # Handlers (ações disparadas por outras tarefas)
│   │   │   └── main.yml
│   │   └── tasks            # Tarefas principais da role
│   │       └── main.yml
│   ├── debian_10_to_latest  # Realizando a atualização do debian 10 para últimas versões de pacotes (apt-get upgrade -y)
│   │   ├── handlers
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   └── debian_11_to_12      # Role para atualizar do Debian 11 para o Debian 12
        ├── files
        │   └── sources.list
        ├── handlers
        │   └── main.yml
        └── tasks
            └── main.yml

```

- **ansible.cfg**: Contém configurações globais para o Ansible, como o caminho para o inventário, a chave SSH padrão, etc.
- **inventory/hosts**: Define os grupos de hosts e os servidores nos quais os playbooks serão executados. É aqui que você listará seus servidores Debian.
- **playbooks/update_debian.yml**: Este é o playbook principal que orquestra a execução das diferentes roles para realizar as atualizações de versão.
- **roles/**: Cada subdiretório dentro de `roles/` representa uma "role" Ansible. Roles são unidades reutilizáveis de automação.
- **handlers/**: Define ações que são acionadas por outras tarefas, como reiniciar serviços.
- **tasks/**: Contém a sequência de tarefas para a atualização (ex: atualização de pacotes, modificação de arquivos de configuração, etc.).
- **files/**:  Diretório que contem arquivos a serem utilizados ou enviados para os servidores em tasks.

> ## (Recomendação: Veja a seção "Recomendações Importantes" para uma discussão sobre esta role).
> Por Que Atualizar versão por versão?
> A decisão de atualizar o Debian versão por versão (10 para 11 e depois para 12) em vez de ir diretamente da 10 para 12 é uma prática altamente recomendada e, em muitos casos, obrigatória. Existem vários motivos cruciais para isso:

- Suporte e Estabilidade: Cada versão do Debian é desenvolvida e testada para ser uma atualização direta da versão anterior. Pular versões pode levar a incompatibilidades de pacotes, bibliotecas desatualizadas, problemas de dependência que não foram previstos e testados para um salto tão grande.
- Transições de Pacotes e Configurações: Durante uma atualização de versão, o Debian lida com transições de pacotes, migração de configurações e, por vezes, alterações significativas em como certos serviços funcionam. A atualização incremental permite que essas transições ocorram de forma mais controlada.
- Depreciação de Software: Softwares e bibliotecas que eram comuns em uma versão podem ter sido removidos ou substituídos em versões subsequentes. Pular versões pode deixar seu sistema em um estado inconsistente onde dependências cruciais estão faltando ou são incompatíveis.
- Resolução de Problemas Facilitada: Se ocorrer um problema durante a atualização, é muito mais fácil diagnosticar e resolver quando o problema é limitado a uma transição de uma versão para a próxima. Ao pular versões, múltiplos pontos de falha potenciais são introduzidos, tornando a depuração exponencialmente mais complexa.
- Documentação e Suporte: A documentação oficial do Debian e a comunidade de suporte geralmente focam em caminhos de upgrade de uma versão para a seguinte. Você encontrará muito mais ajuda e recursos se seguir esse caminho.

Em resumo, atualizar versão por versão é a abordagem mais segura, estável e suportada para manter a integridade do seu sistema Debian.

## Pré-requisitos
Antes de executar este projeto Ansible, certifique-se de que os seguintes pré-requisitos sejam atendidos:

- Ansible Instalado: Você precisa ter o Ansible instalado na sua máquina de controle (a máquina de onde você executará os playbooks).
- Acesso SSH sem senha: O usuário Ansible na sua máquina de controle deve ter acesso SSH sem senha aos servidores Debian alvo. Recomenda-se o uso de chaves SSH.
- Permissões de sudo: O usuário que o Ansible utiliza nos servidores remotos deve ter permissões de sudo para executar comandos com privilégios.
- Conectividade de Rede: Os servidores Debian alvo devem ter acesso à internet para baixar os pacotes de atualização dos repositórios oficiais do Debian.
- Backups: É ABSOLUTAMENTE CRÍTICO FAZER BACKUPS COMPLETOS DE TODOS OS SEUS DADOS E CONFIGURAÇÕES ANTES DE INICIAR QUALQUER PROCESSO DE ATUALIZAÇÃO DE SISTEMA OPERACIONAL. Em caso de falha, um backup é sua única garantia de recuperação.


## Como Utilizar
Siga os passos abaixo para utilizar este projeto Ansible para atualizar seus sistemas Debian.

Clone o Repositório:

```Bash
git clone https://github.com/GabrielHespanhol/ansible-debian-upgrade.git
cd ansible-debian-upgrade
```

Configure o Inventário (inventory/hosts):

Edite o arquivo `inventory/hosts` para listar seus servidores Debian. Você pode definir grupos para organizar seus hosts. A seguir um exemplo onde temos o grupo debian_servers e dois servidores, sendo:
- servidor_debian_01 e servidor_debian_02 um "nome/apelido".
- ansible_host=192.168.1.100 o endereço de ip do servidor remoto que vamos conectar.
- ansible_user=seu_usuario_ssh o usuário que vamos utilizar para conectar no servidor remoto.
- ansible_port=2222 pode ser utilizado para definir a porta de conexão ssh, por padrão é a 22.

## Configuração do arquivo de inventario de hosts
```Ini, TOML
[debian_servers]
servidor_debian_01 ansible_host=192.168.1.100 ansible_user=seu_usuario_ssh ansible_port=2222
servidor_debian_02 ansible_host=192.168.1.101 ansible_user=seu_usuario_ssh ansible_port=2022
```

A seguir um comando de exemplo para executar o playbook, onde a opção -K vai solicitar a senha de root do servidor remoto, utilizada para execução de tarefas como administrador.

## Executando o playbook
```bash
ansible-playbook playbooks/update_debian.yml -l teste -K
```

## Arquivo ansible.cfg

Esse arquivo tem configurações globais do ansible, como por exemplo o usuário padrão de conexão, diretório com a chave ssh, entre outras possíveis opções não listadas aqui. Recomendo fazer uma leitura da documentação. 