# Cenário do desafio

Você faz estágio na área de engenharia da nuvem em uma nova startup.
Como seu primeiro projeto, pediram que você criasse uma infraestrutura
de maneira rápida e eficiente, além de gerar um mecanismo de acompanhamento
para consultas e mudanças futuras. Você recebeu orientações para usar o 
Terraform na realização dessa tarefa.

Nesse projeto, você vai usar o Terraform para criar, implantar e acompanhar
a infraestrutura no servidor preferido da startup, o Google Cloud. Você também
vai precisar importar algumas instâncias com problemas de gerenciamento para
sua configuração e fazer as correções necessárias.

Neste laboratório, você vai usar o Terraform para importar e criar várias
instâncias de VM, uma rede VPC com duas sub-redes e uma regra de firewall
para a VPC que permita conexões entre as duas instâncias. Você também vai
criar um bucket do Cloud Storage para hospedar seu back-end remoto.

## Tarefa 1: crie os arquivos de configuração

1️⃣ No Cloud Shell, crie seus arquivos de configuração do Terraform e uma estrutura de diretórios como esta:
```
main.tf
variables.tf
modules/
└── instances
    ├── instances.tf
    ├── outputs.tf
    └── variables.tf
└── storage
    ├── storage.tf
    ├── outputs.tf
    └── variables.tf
```
2️⃣ Preencha os arquivos variables.tf no diretório raiz e nos módulos.
Adicione três variáveis para cada arquivo: ```region```, ```zone``` e ```project_id```.
Como valores padrão, use us-east1, us-east1-c e seu ID do projeto do Google Cloud.

3️⃣ Adicione o bloco do Terraform e o provedor do Google (link em inglês) ao arquivo main.tf.
Verifique se o argumento de zona foi adicionado com os argumentos de projeto e região no bloco do provedor do Google.

4️⃣ Inicialize o Terraform.

## Tarefa 2: importe a infraestrutura

1️⃣ No Console do Google Cloud, no Menu de navegação, clique em Compute Engine > Instâncias de VM.
Duas instâncias chamadas tf-instance-1 e tf-instance-2 já foram criadas para você.

2️⃣ Importe as instâncias atuais para o módulo instances. Para isso, siga estas etapas:

Primeiro, adicione a referência do módulo ao arquivo main.tf e reinicialize o Terraform.
Em seguida, escreva as configurações do recurso no arquivo instances.tf para corresponder às instâncias atuais.

Dê às instâncias os nomes ```tf-instance-1``` e ```tf-instance-2```.
Neste laboratório, a configuração do recurso deve ser mínima.
Para isso, você vai precisar incluir estes argumentos extras na configuração:
```machine_type```, ```boot_disk```, ```network_interface```, ```metadata_startup_script``` e ```allow_stopping_for_update```.
No caso dos últimos dois argumentos, use a configuração a seguir para que não seja necessário recriá-la:
```
metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
allow_stopping_for_update = true
```
Depois de escrever as configurações do recurso no módulo, 
use o comando terraform import e importe essas preferências para o módulo instances.

3- Aplique as alterações. Como você não preencheu todos os argumentos na configuração, 
o código apply vai atualizar as instâncias atuais. Essa opção é aceita no laboratório, 
mas é necessário preencher todos os argumentos corretamente antes da importação em um ambiente de produção.

## Tarefa 3: configure um back-end remoto

1- Crie um recurso de bucket do Cloud Storage no módulo storage.
Para nomear o bucket, use tf-bucket-377907.
Para os outros argumentos, use estes valores:

location = "US"
force_destroy = true
uniform_bucket_level_access = true

2- Adicione a referência do módulo ao arquivo main.tf.
Inicialize o módulo e aplique (apply) as mudanças para criar o bucket usando o Terraform.

3- Configure o bucket de armazenamento como o back-end remoto no arquivo main.tf.
Para possibilitar a avaliação, use o prefixo terraform/state.

4- Se a configuração estiver correta, após o comando init, 
o Terraform vai perguntar se você quer copiar os dados de estado para o novo back-end.
Digite yes no prompt.

## Tarefa 4: modifique e atualize a infraestrutura

1- Acesse o módulo instances e modifique o recurso tf-instance-1 para usar um tipo de máquina e2-standard-2.

2- Altere o recurso tf-instance-2 para usar um tipo de máquina e1-standard-2.

3- Adicione um terceiro recurso de instâncias chamado tf-instance-999986. Use um tipo de máquina e2-standard-2 para ele. Mude o tipo de máquina para e2-standard-2 nas três instâncias.

4- Inicialize o Terraform e aplique (apply) as mudanças.

## Tarefa 5: destrua os recursos

Remova o recurso do arquivo de configuração para destruir a terceira instância tf-instance-999986.
Depois disso, inicialize o Terraform e aplique (apply) as mudanças.

## Tarefa 6: use um módulo do Registry

1- No Terraform Registry, procure o módulo network (em inglês).

2- Adicione o módulo ao arquivo main.tf. Use as configurações a seguir:

Utilize a versão 6.0.0. Outras versões podem gerar erros de compatibilidade.
Nomeie a VPC como tf-vpc-385209 e use o modo de roteamento global.
Especifique 2 sub-redes na região us-east1 chamadas de subnet-01 e subnet-02. Para os argumentos de sub-rede, é necessário incluir um Nome, um IP e uma Região.
Use o IP 10.10.10.0/24 para subnet-01 e 10.10.20.0/24 para subnet-02.
Como essa VPC não exige intervalos secundários ou rotas associadas, omita essas opções da configuração.

3- Depois de escrever a configuração do módulo, inicialize o Terraform e execute a aplicação (apply) para criar as redes.

4- Em seguida, acesse o arquivo instances.tf e atualize os recursos de configuração para conectar tf-instance-1 a subnet-01 e tf-instance-2 a subnet-02.

Observação: nessa configuração de instância, você vai precisar atualizar o argumento de rede para tf-vpc-385209 e adicionar o argumento de sub-rede com a sub-rede certa para cada instância.

## Tarefa 7: configure um firewall

Crie um recurso de regra de firewall chamado tf-firewall no arquivo main.tf.

Essa regra de firewall deve permitir que a rede tf-vpc-385209 autorize conexões de entrada em todos os intervalos de IP (0.0.0.0/0) na porta TCP 80.
Adicione o argumento source_ranges com o intervalo de IP correto (0.0.0.0/0).
Inicialize o Terraform e aplique (apply) as mudanças.

`Observação: para recuperar o argumento network obrigatório, 
inspecione o estado e encontre o ID ou o self_link do recurso google_compute_network que você criou.
Essa informação fica no formulário projects/PROJECT_ID/global/networks/tf-vpc-385209.`
