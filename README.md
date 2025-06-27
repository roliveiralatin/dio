
# Monitoramento e Resposta Proativa a Eventos Críticos no Azure  

## 📘 Visão Geral
Este documento descreve boas práticas e ferramentas para manter visibilidade, controle e resposta proativa frente à exclusão de uma VM no Microsoft Azure, garantindo a continuidade operacional e mitigando riscos de indisponibilidade ou perda de dados.

---

## 🔍 Visibilidade

### 1. Azure Activity Log
- O que é: Registro centralizado de operações realizadas em nível de assinatura.
- Utilização:
  - Rastrear operações de exclusão (`Delete`) em VMs.
  - Identificar o usuário, data/hora, origem IP e recurso afetado.
- Ação:
  - Navegar até Monitor > Activity Log e aplicar filtros como:
    - Operation Name: `Delete Virtual Machine`
    - Status: `Succeeded`
    - Resource Group ou VM específica

### 2. Log Analytics + Azure Monitor
- Permite consultas detalhadas com Kusto Query Language (KQL).
- Exemplo de query para identificar exclusão de VMs:
  ```kql
  AzureActivity
  | where OperationNameValue == "Microsoft.Compute/virtualMachines/delete"
  | sort by TimeGenerated desc
  ```

### 3. Azure Policy
- Aplicação de políticas de governança para bloquear ou auditar exclusões indesejadas.

---

## 🛡️ Controle

### 1. RBAC (Role-Based Access Control)
- Princípio do menor privilégio: apenas usuários autorizados devem ter permissões para deletar VMs.
- Ações recomendadas:
  - Auditar e revisar permissões periodicamente.
  - Evitar uso de `Owner` desnecessário.
  - Criar papéis customizados sem permissão `Microsoft.Compute/virtualMachines/delete`.

### 2. Resource Locks (Bloqueios de Recursos)
- Aplicar `CanNotDelete` em VMs críticas:
  ```bash
  az lock create     --name "lock-anti-delete"     --lock-type CanNotDelete     --resource-group MeuGrupo     --resource MinhaVM     --resource-type "Microsoft.Compute/virtualMachines"
  ```

### 3. Políticas de Naming e Tagging
- Padronizar nomes e utilizar tags (`environment=prod`, `critical=yes`) para facilitar a aplicação de políticas e alertas.

---

## 🚨 Resposta Proativa

### 1. Alertas em Tempo Real
- Criar alertas baseados no `Activity Log` para operações de exclusão:
  - Signal: `Administrative`
  - Operation: `Delete Virtual Machine`
  - Action Group: enviar email, webhook, ou chamar função do Azure

### 2. Automação com Azure Logic Apps / Functions
- Reagir automaticamente a eventos de exclusão:
  - Enviar alertas via Teams, Slack ou SMS
  - Disparar playbook de investigação
  - Criar incidentes no ITSM (ex: ServiceNow)

### 3. Backup e Recuperação
- Garantir que Azure Backup esteja habilitado.
- Restaurar a VM em caso de exclusão acidental:
  - Via portal ou CLI:
    ```bash
    az backup restore restore-disks       --vault-name MeuCofre       --resource-group MeuGrupo       --container-name vmname       --item-name vmname       --restore-to-staging-storage-account
    ```

---

## ✅ Boas Práticas Recomendadas

- Ativar Azure Defender for Cloud para recomendações e alertas de segurança.
- Auditar regularmente as permissões (RBAC) e eventos com ferramentas como Microsoft Sentinel.
- Definir um Runbook de resposta para incidentes envolvendo recursos críticos.

---

## 📎 Recursos Úteis

- [Azure Activity Log Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/activity-log)
- [Azure RBAC Best Practices](https://learn.microsoft.com/en-us/azure/role-based-access-control/best-practices)
- [Lock resources to prevent changes](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources)
- [Azure Monitor Alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview)
- [Azure Backup Documentation](https://learn.microsoft.com/en-us/azure/backup/)

---

## 📌 Conclusão
A exclusão de uma VM no Azure pode ter impactos críticos. Manter visibilidade com logs, aplicar controles com RBAC e bloqueios, além de configurar alertas e automações, são práticas fundamentais para ambientes corporativos em nuvem. A prevenção é a chave para a segurança e continuidade operacional.
