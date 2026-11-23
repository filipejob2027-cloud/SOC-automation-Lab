## Triagem de alertas (simulação de L1)

### Simulação

No PowerShell como administrador, criei uma conta e promovi-a a administrador, e corri comandos de reconhecimento:

```powershell
net user labtest <palavra-passe-de-teste> /add
net localgroup Administradores labtest /add
```

### Alerta

![Alerta 60154 no Wazuh](img/wazuh-alerta-60154.png)

| Campo | Valor |
|---|---|
| Regra | 60154, Administrators Group Changed |
| Nível | 12 |
| Técnica MITRE | T1484 |
| Evento do Windows | 4732 |
| Computador | win-lab |
| Quem fez | epcps |
| Grupo alterado | Administradores |
| Quem entrou | `S-1-5-21-1105318813-943618807-4205460125-1018` (conta `labtest`) |
| Hora | 23/11/2026, 00:29 |

### Triagem

### Triagem

- **O que foi detetado:** A conta `labtest` (`S-1-5-21-1105318813-943618807-4205460125-1018`) foi adicionada ao grupo `Administradores` no computador `win-lab` pela conta `epcps`, gerando o evento do Windows `4732` e o alerta da regra `60154` (`Administrators Group Changed`, nível 12, técnica MITRE `T1484`) às 00:29 do dia 23/11/2026.
- **É esperado?** Sim, porque a ação foi executada por mim no laboratório (`epcps`) para testar a deteção de alterações a grupos privilegiados.
- **Risco se não a reconhecesse:** Se não tivesse sido autorizada, significaria que uma conta com privilégios administrativos estava a criar ou a promover uma nova conta para obter acesso administrativo persistente no sistema.
- **Decisão:** Atividade autorizada de laboratório (não é um incidente real). Num ambiente real, trataria este alerta de nível 12 com prioridade elevada até confirmar junto da administração/IT se a alteração era legítima.
- **Próximos passos:** Validar se não houve mais ações inesperadas na máquina `win-lab`, remover a conta de teste `labtest` quando o teste terminar e registar a conclusão da triagem no caso do TheHive.


### Caso no TheHive

<Abri um caso com ... e registei ...>

![Caso no TheHive](img/thehive-caso-labtest.png)

### O que mostra este exercício

<O que concluíste? O que aprendeste sobre alertas e níveis?>
