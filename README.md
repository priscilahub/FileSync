# Exercicio: 1- Prepare o ambiente de Server Endpoint 

### Atividade 1 - Implante uma VM do Azure 
Use um modelo ARM para criar uma VM com um disco de dados e simular um servidor local.

1. Logue no **Azure Portal.**
2. Acesse o [Link](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-dynamic-data-disks-selection), para implementar um servidor windows.
3. Adicione um user name e uma senha.
4. Em number of vms e number of disks os valores são: 1
5. Os demais parâmetros podemos manter como estão.


### Atividade 2 - Prepare o Servidor para usar com o Azure File Sync. 

1. No portal do Azure navegue até a **VM** criada.
2. Conecte-se a ela via RDP.
3. Quando for solicitado a fazer login, forneça as credenciais criadas no exercício anterior.
4. Adicione o disco de dados no servidor navegando até o gerenciamento de discos. 
5. Crie uma nova partição GPT e adicione um novo volume NTFS com compartilhamento F:\
6. No File Explorer navegue até a partição criada, adicione uma pasta chamada **Azure** e dentro dela adicione **arquivos txt** para testes futuros.

### Atividade 3 – Instale o mais recente módulo Az PowerShell e realize um teste de compatibilidade.

1. Inicie o PowerShell como administrador.
2. Execute o seguinte commando para instalar o **módulo do Az do powershell**:

`Install-Module -Name Az -AllowClobber -SkipPublisherCheck`

3. Verifique se a unidade **F:\Azure**  criada recentemente possui algum problema de compatibilidade com o Azure File Sync executando o comando: 

`Invoke-AzStorageSyncCompatibilityCheck -Path 'F:\Azure'`

4. As respostas do comando devem trazer os seguintes resultados para confirmar a compatibilidade: 
```
OS version check: Passed. 
File system check: Passed.
```
### Atividade: 3 - Instale o agente de sincronização 

1. Navegue até a Local Server e desligue temporariamente a **configuração de segurança aprimorada do IE**.
2. Inicie o Internet Explorer e navegue até o Microsoft Download Center em https://go.microsoft.com/fwlink/?linkid=858257 
3. Baixe o arquivo  **Azure File Sync Agent Windows Installer** versão 2019.
4. Uma vez que o download seja concluído, execute o assistente de configuração do agente com as configurações padrões para instalar o **Azure File Sync Agent**.
5. Após a instalação do agente Azure File Sync ser concluída, o assistente será iniciado automaticamente.

# Exercicio: 2- Prepare o ambiente de Cloud endpoint

### Atividade 1 – Crie um Storage Account com um Azure File share utilizando ARM template: 

1. Copie o código abaixo e cole no gerenciador de deployments customizados no Azure.
2. Insira um nome para o Storage Account e para o Azure FileShare.

 
```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "metadata": {
    "_generator": {
      "name": "bicep",
      "version": "0.4.1008.15138",
      "templateHash": "3369765991499277236"
    }
  },
  "parameters": {
    "storageAccountName": {
      "type": "string",
      "defaultValue": "",
      "metadata": {
        "description": "Specifies the name of the Azure Storage account."
      }
    },
    "fileShareName": {
      "type": "string",
      "maxLength": 63,
      "minLength": 3,
      "metadata": {
        "description": "Specifies the name of the File Share. File share names must be between 3 and 63 characters in length and use numbers, lower-case letters and dash (-) only."
      }
    }
  },
  "functions": [],
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "kind": "StorageV2",
      "sku": {
        "name": "Standard_LRS"
      },
      "properties": {
        "accessTier": "Hot"
      }
    },
    {
      "type": "Microsoft.Storage/storageAccounts/fileServices/shares",
      "apiVersion": "2021-04-01",
      "name": "[format('{0}/default/{1}', parameters('storageAccountName'), parameters('fileShareName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]"
      ]
    }
  ]
}

```
> caso o Azure fileshare não seja implementado automaticamente, sugiro adiciona-lo manualmente pelo portal.

# Exercicio: 3- Configure a sincronização de Arquivos 

### Atividade: 1 – Configure o Storage Sync Service
1. No portal do Azure busque por **Storage Sync Services** clique em create.
2. Selecione a mesma subscription e região utilizada para configurar o storage account.
3. Insira um nome e clique em create.

### Atividade: 2 – Configure o Cloud Endpoint
1. Navegue até o **Storage Sync Service** clique no campo campo **Sync Group** e crie um novo grupo de sincronização com as seguintes configurações:
	- Nome do grupo de sincronização.
	- O nome da assinatura que você está usando neste laboratório
	- Conta de armazenamento que você criou no exercício anterior
	- Azure File Share criado na conta de armazenamento

### Atividade: 3 – Configure o Server Endpoint
1. Registre o servidor Windows criado no início do laboratório com o **Storage Sync Service**, acessando o servidor criado no início do laboratório.
2. Busque por **Azure Storage Sync Agent** e o agente de configuração será exibido, nele você deve realizar as seguintes configurações:
  - Autentique sua conta do Azure selecionando a opção **AzureCloud**.
  - Adicione a subscription, que utilizada no laboratório.
  - Adicione o **Resource Group** e o **Storage Sync Service** criado recentemente.
  - Clique em registrar e verifique a seguinte resposta do agente: **Registration sucessful**.
 3. Adicione o servidor Windows ao Server endpoint
  - No portal navegue até **Storage Sync Service**. 
  - No menu lateral busque por **Registered servers** e verifique se o status do mesmo é **Online**.
  - Agora busque pelo **Sync Group** criado anteriormente clique no mesmo.
 4. Dentro do Sync Group criado, clique em **add server endpoint** e uma nova blade abrirá solicitando as seguintes configurações:
  - Selecione o servidor recentemente registrado.
  - No nome do path solicitado adicione o path criado no servidor local: **F:\Azure**
  - Adicione o Server Endpoint.

### Atividade: 4 – Avalie as atividades de sincronismo
 - Acesse o **Azure Files** no storage account.
 - Verifique se os mesmos arquivos de testes criados na pasta F:\Azure foram sincronizados.

Pronto! Agora você tem arquivos locais sincronizando com o Azure File Sync para um Azure File Share não é legal?


>Referências
>https://docs.microsoft.com/en-us/azure/templates/microsoft.compute/virtualmachines?tabs=bicep
>https://docs.microsoft.com/en-us/azure/templates/microsoft.storage/storageaccounts?tabs=bicep
>https://docs.microsoft.com/en-us/azure/storage/file-sync/file-sync-introduction

