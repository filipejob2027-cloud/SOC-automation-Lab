# SOC Homelab

Este homelab foi criado para eu perceber, de forma prática, como é o trabalho num SOC. Simula o fluxo de um analista: o Sysmon regista a atividade do Windows, o Wazuh recolhe e analisa esses eventos para gerar alertas, e o TheHive serve para investigar e documentar casos. Gero atividade suspeita no meu próprio computador para estudar os alertas e treinar a triagem, tudo isolado em contentores Docker.

 

## 1. Sysmon

Usei o Sysmon porque é uma boa ferramenta para ver o que normalmente o Windows não regista, nomeadamente:

- Processos criados
- Ligações de rede
- Ficheiros criados
- Consultas de DNS
- Alterações de registo
O Sysmon apenas regista, não intervém nem alerta.

### Configuração
Configuração baseada em [SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config) (v74), licença CC BY 4.0.

Como decide o que regista
•	Cada tipo de evento tem regras include (só regista o que bate certo) ou exclude (regista tudo, menos o que bate certo).
•	Nos processos criados (ProcessCreate), a configuração usa exclude: regista todos, menos uma lista de ruído conhecido do Windows.
•	Nas ligações de rede (NetworkConnect), usa include: só regista as de programas ou portas interessantes.
•	O objetivo é filtrar o ruído no próprio computador, para os eventos úteis se verem melhor.

###Verificação 
 


## 2. Wazuh

O Wazuh é uma plataforma de deteção de ameaças. Recebe os eventos que o Sysmon regista no Windows, analisa-os com regras e gera um alerta quando algo parece suspeito. 
Funciona com duas partes: um **agente**, instalado no computador vigiado, e um **manager**, o servidor central.

### Como os eventos chegam ao Wazuh

1. O Sysmon regista a atividade do Windows e escreve cada evento no canal `Microsoft-Windows-Sysmon/Operational` do log de eventos.
2. O agente Wazuh lê esse canal. Isto está definido no `ossec.conf` do agente:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

3. O agente envia os eventos ao manager. Não decide nada: só lê e envia.
4. O manager aplica as regras. Quando uma regra bate certo, gera um **alerta**.
5. Os alertas ficam guardados no indexer e consultam-se no dashboard.

### Componentes

O Wazuh (versão 4.9.0) corre em Docker, com 3 contentores:

| Componente | Função |
|---|---|
| wazuh-manager | Recebe os eventos dos agentes, analisa-os com regras e gera os alertas |
| wazuh-indexer | Guarda os eventos e alertas de forma pesquisável |
| wazuh-dashboard | Interface web onde se veem e pesquisam os alertas |

### Portas

| Porta | Para que serve |
|---|---|
| 8443 | Dashboard (`https://localhost:8443`) |
| 1514 e 1515 | Comunicação com os agentes (eventos e registo) |
| 5555 | API do Wazuh (55000 dentro do contentor) |

### Como arrancar

```powershell
cd C:\Wazuh-SOC
docker compose up -d
```

O indexer demora alguns minutos a ficar pronto. Enquanto isso, o dashboard mostra "server is not ready yet".

### Verificação

No dashboard, o agente aparece como **Active** (Agents Summary).

 

## 3. TheHive

O TheHive é onde o analista de SOC abre um caso a partir de um alerta, o investiga e documenta.
O Wazuh deteta e gera o alerta. O TheHive é onde o analista abre um caso para o investigar e documentar.

### Componentes

O TheHive (versão 5.2.0) corre em Docker, com 3 contentores:

| Componente | Função |
|---|---|
| thehive | A aplicação e a interface web, onde o analista gere os casos |
| cassandra | A base de dados onde ficam guardados os casos e os seus dados |
| elasticsearch | O motor de pesquisa, que indexa os dados para se poder procurar neles |

### Configuração inicial

A conta inicial do TheHive é de administrador: só serve para criar organizações, utilizadores e permissões. Por isso criei a organização SOC-Lab e um utilizador com o perfil analyst, que é quem abre e trabalha os casos.
- Organização: `SOC-Lab`
- Utilizador: perfil `analyst`

### Portas

| Porta | Para que serve |
|---|---|
| 9000 | Interface web (`http://localhost:9000`) |
| 9201 (9200 no contentor) | Elasticsearch, mudada de 9200 por conflito com o indexer do Wazuh |

### Como arrancar

```powershell
docker compose -f "C:\WINDOWS\system32\thehive-lab\docker-compose.yml" up -d
```
Nota: por opção durante a montagem, o ficheiro ficou em System32, mas o ideal é mantê-lo numa pasta dedicada fora do sistema (ex.: C:\Labs\thehive-lab).

### Como se liga ao Wazuh

A ligação é manual , abrimos o TheHive localmente para estudarmos o caso.
### Verificação
Teste
 


## 4. Docker

O Docker é uma plataforma de contentorização: permite correr aplicações dentro de contentores, que são ambientes isolados com tudo o que a aplicação precisa para funcionar.

Usei Docker em vez de instalar tudo no Windows porque assim cada ferramenta fica isolada, instala-se com um comando e é fácil apagar e refazer o lab. Também é mais leve do que montar uma máquina virtual para cada componente.
### O que corre em Docker

| Projeto | Contentores | Onde está o ficheiro |
|---|---|---|
| Wazuh | wazuh-manager, wazuh-indexer, wazuh-dashboard | `C:\Wazuh-SOC\docker-compose.yml` |
| TheHive | thehive, cassandra, elasticsearch | `<pasta do TheHive>\docker-compose.yml` |

### Como funciona o docker-compose

O docker-compose.yml descreve os contentores de um projeto e como se ligam entre si. O do Wazuh define o manager, o indexer e o dashboard. 
O comando docker compose up -d lê o ficheiro, cria os contentores e arranca-os em segundo plano

### Comandos que usei

| Comando | Para que serve |
|---|---|
| `docker ps` | Lista os contentores que estão a correr, com o estado e as portas. Com -a mostra também os parados. |
| `docker compose up -d` | Cria e arranca os contentores do projeto, em segundo plano.|
| `docker compose down` | Para e apaga os contentores e a rede. Sem -v, os dados (volumes) mantêm-se.|
| `docker logs <contentor>` | Mostra o que o contentor escreveu, útil para perceber porque falha (--tail 30 mostra só as últimas 30 linhas).|
| `docker stop <contentor>` | Para o contentor sem o apagar, e fica pronto a arrancar de novo.|

### Requisitos

O Docker Desktop tem de estar aberto e em "Engine running". Foi o que falhou no início, quando o docker version deu erro.
No Windows, o Docker corre os contentores por cima do WSL2, por isso o Docker Desktop tem de estar a correr.
O lab pesa: são 6 contentores. Por isso arranquei o Wazuh e o TheHive em separado.

### Verificação

 

