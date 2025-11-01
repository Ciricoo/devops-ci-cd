# Projeto DevOps: Automação de CI/CD com GitLab CE

Este é um trabalho acadêmico desenvolvido para a matéria de GERENCIAMENTO, CONFIGURAÇÃO E PROCESSOS DE SOFTWARE, que apresenta um processo completo de integração e entrega contínua (CI/CD) baseado em um conjunto de ferramentas open-source e com hospedagem própria (self-hosted).

A meta é replicar um cenário de produção onde o código-fonte é controlado, compilado (build) e entregue (deploy) de maneira totalmente automatizada.

## 1. Integrantes

- João Vitor Círico

## 2. Ferramentas Adotadas

- **Aplicação:** API REST minimalista (Node.js + Express).
- **Plataforma Integrada:** GitLab Community Edition (CE).
- **Agente de CI/CD:** GitLab Runner.
- **Infraestrutura:** Docker e Docker Compose.

### Justificativa das Escolhas

O **GitLab Community Edition (CE)** foi selecionado como a plataforma DevOps central desta stack.

1.  **Gratuito e Self-Hosted:** Cumpre a exigência de não depender de serviços SaaS.
2.  **Plataforma Consolidada ("All-in-One"):** O maior benefício estratégico. O GitLab CE resolve as três principais necessidades do projeto em um único pacote:
    - **Controle de Versão:** Servidor Git completo (alternativa ao Gitea).
    - **Automação (CI/CD):** Sistema de pipeline nativo (GitLab CI) configurado via `.gitlab-ci.yml`.
    - **Registro de Artefatos:** Um "Container Registry" para imagens Docker, já integrado.

## 3. Desenho do Fluxo de Automação

O processo de automação configurado opera da seguinte forma:

`[Developer]` -> `git push` -> `[GitLab CE]` -> `[GitLab Runner (DooD)]` -> `docker build` -> `docker push` -> `[GitLab Container Registry]`

## 4. Guia de Instalação do Ambiente

O projeto está organizado em dois diretórios principais:

1.  `infra`: Contém o `docker-compose.yml` para provisionar a infraestrutura (GitLab CE e GitLab Runner).
2.  `api`: Inclui o código da aplicação Node.js, seu `Dockerfile` e o script `.gitlab-ci.yml`.

### Pré-requisitos

- Docker e Docker Compose
- Uma máquina com pelo menos 8GB de RAM (Recomendado)

### 4.1. Iniciando a Infraestrutura

1.  **Provisionar os contêineres (GitLab + Runner):**

    ```bash
    cd infra
    docker compose up -d
    ```

    *(Aguarde alguns minutos para a inicialização completa do GitLab.)*

2.  **Ajustar o DNS Local:**
    Adicione a seguinte linha ao seu arquivo `/etc/hosts`:

    ```
    127.0.0.1   gitlab.local
    ```

3.  **Primeiro Acesso ao GitLab:**

    - Acesse `http://gitlab.local` no seu navegador.
    - Obtenha a senha de `root` inicial: `docker compose exec -it gitlab cat /etc/gitlab/initial_root_password`
    - Faça login com o usuário `root` e a senha obtida, depois defina uma nova senha.

### 4.2. Configurando e Registrando o Runner

1.  No painel "Admin Area" do GitLab (`http://gitlab.local`), acesse **CI/CD** > **Runners**.
2.  Clique no botão azul **New instance runner**.
3.  Adicione a tag `docker` e clique em **Create runner**.
4.  Na página seguinte, **copie o token de autenticação** (ex: `glrt-...`).

5.  Utilize o token para gerar o arquivo de configuração do Runner. No terminal (na pasta `infra`):

    ```bash
    # (Verifique se está na pasta infra)
    cd infra

    # Copie o arquivo de exemplo para um arquivo .toml temporário
    cp config-runner.toml-exemplo config.toml

    # Abra o novo arquivo .toml para editá-lo
    nano config.toml
    ```

6.  Dentro do `nano`, encontre a linha `token = "COLE_O_TOKEN_GERADO_PELA_UI_AQUI"`.
7.  **Substitua o placeholder `"COLE_O_TOKEN..."` pelo seu token (`glrt-...`)** que você copiou da UI.
8.  Salve e feche (Ctrl+O, Enter, Ctrl+X).

9.  Copie o arquivo de configuração final para o contêiner do Runner (este passo efetiva o registro):

    ```bash
    docker cp config.toml gitlab-runner:/etc/gitlab-runner/config.toml

    # Reinicie o Runner para aplicar a nova configuração
    docker compose restart gitlab-runner

    # (Opcional) Remova o arquivo temporário
    rm config.toml
    cd ..
    ```

10. **Conferência:** Retorne à "Admin Area > Runners" no GitLab. O Runner deve aparecer online (círculo verde 🟢) em instantes.

### 4.3. Enviando a Aplicação

1.  Crie um "New project" vazio no GitLab (ex: `devops`).
2.  Navegue até o diretório `api`:
    ```bash
    cd api
    ```
3.  Use os comandos Git para enviar o código-fonte para seu **GitLab local**:
    ```bash
    git init
    git remote add origin [http://gitlab.local/root/lab-devops.git](http://gitlab.local/root/lab-devops.git)
    git add .
    git commit -m "Commit inicial"
    git push -u origin main
    ```

*O pipeline será acionado automaticamente após o push.*

## 5. Material de Apresentação

O material de apoio (slides) usado na apresentação está no Canva:
[Link para os Slides da Apresentação](https://www.canva.com/design/DAG3auS-gNQ/2ISkHk-TOMnYFnzXbasSmA/view?utm_content=DAG3auS-gNQ&utm_campaign=designshare&utm_medium=link2&utm_source=uniquelinks&utlId=h8a5fc92a49)