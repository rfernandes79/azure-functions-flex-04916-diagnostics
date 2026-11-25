# Azure Functions Flex Consumption — diagnóstico público do erro 04916

Este repositório reúne uma versão anonimizada das informações técnicas para discussão com a comunidade Azure. Os arquivos descrevem o caso observado e os limites da evidência disponível.

## Erro recebido

```text
Deployment for site '[APP_NAME]' with DeploymentId '[DEPLOYMENT_ID]' failed because the operation timed out after '20' minutes. Please try the deployment again.
```

Os identificadores do aplicativo e da implantação foram substituídos por aliases. O texto restante foi preservado. Foram reportados `RuntimeFailed`, `ExpectationFailed`, código `04916`, timeout de 20 minutos e contadores de instâncias zerados. Esses zeros não comprovam validação bem-sucedida.

## Implantação e ambiente

- Azure CLI, comando `az functionapp deployment source config-zip`, executado pelo operador no Cloud Shell Bash por meio de um roteiro Python.
- Remote build desativado; um único envio controlado, sem retry ou rollback.
- Node.js22, Linux, Flex Consumption, região Brazil South; 18 funções HTTP.
- O host observado usa extensão `~4`, versão `4.1055.300.26527` no exemplo de log.
- O [README em inglês](README.md) contém o comando de referência com placeholders e a descrição detalhada.

## Rede, identidade e permissões consultadas

A configuração retornou o campo de subnet VNet sem referência configurada. Host e implantação selecionam a mesma identidade gerenciada atribuída ao aplicativo e a mesma conta Storage. Foram listados Blob Data Owner no host e Blob Data Contributor no contêiner de implantação, além de Blob Data Owner herdado, sem condições nas atribuições reportadas. O Storage aparece com acesso de rede Enabled e firewall padrão Allow.

**Private Endpoint e regras de acesso por IP/SCM não foram conferidos.** A leitura de configuração/IAM não testou conectividade real, permissões efetivas, grupos, deny/PIM ou saúde de todas as instâncias, nem atesta retroativamente a configuração durante a implantação.

## Logs e interpretação

Os [45 eventos Kudu/FDM](logs/deployment-kudu-fdm.log) mostram validação/extração, pós-build e upload concluídos, seguidos de mensagem de conclusão do pipeline FDM. O [exemplo do host](logs/host-startup.sample.log) inclui `18 functions loaded`, geração de18 jobs e início do host. É um exemplo de uma exportação parcial, sujeita a amostragem; não comprova ausência de erros em toda a janela.

A causa do timeout continua não demonstrada. Também não está comprovado se ocorreu implantação pelo caminho de fallback descrito na documentação. O objetivo da discussão é identificar o diagnóstico específico que esclarece essa distinção e o ponto de validação pendente.

## Anonimização

Aplicativos, hosts, IDs de implantação/correlação/instância e nomes de funções foram substituídos por aliases consistentes. Os arquivos públicos contêm apenas a seleção técnica preparada; a aplicação e seus dados de negócio não integram este repositório. Veja as [notas da evidência](EVIDENCE.md).

Referências: [documentação do Flex](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan#deployment-validation-for-code-package-apps) e [triagem pública11952](https://github.com/Azure/azure-functions-host/issues/11952). A triagem sugere análise da plataforma e não estabelece causa comum ou correção para este caso.
