# diagram
<details>
<summary><b>Fase 1: Validación de Identidad y Autorización</b></summary>

```mermaid

sequenceDiagram
    autonumber
    
    actor Dev as Usuario Solicitante 
    participant Teams as Canal Teams
    participant Bot as Azure Bot Service (Enrutador)
    participant Backend as Bot Backend
    participant Agent as AI Foundry Agent
    participant MCP as MCP Server (Tools)
    participant AAD as Azure AD
    participant CVT as Portal App CVT
    participant TFC as Terraform Cloud

    Dev->>Teams: Solicita aprovisionar recurso Cloud
    Teams->>Bot: Envía mensaje
    Bot->>Backend: Enruta payload
    Backend->>Agent: Envía texto (Sesión propagada)
    Agent->>Backend: Pregunta aplicación impactada
    Backend->>Teams: Renderiza Tarjeta Adaptativa
    Teams-->>Dev: ¿Cuál es la app impactada?
    Dev->>Teams: Responde app (cod 4 digitos)
    
    rect rgba(0, 120, 215, 0.1)
        Note right of Agent: Validaciones de Autorización Vía MCP
        Agent->>MCP: Solicita validaciones
        MCP->>CVT: GET /apps/YAPE
        CVT-->>MCP: {exists: true, owner: "po_user@empresa.com", ...}
        MCP->>AAD: Valida ID y Grupos
        MCP->>TFC: Valida Team TFC
        MCP-->>Agent: JSON Enriquecido
    end
    
    alt Usuario NO Autorizado
        rect rgba(255, 0, 0, 0.15)
            Agent->>Backend: Genera mensaje de rechazo
            Teams-->>Dev: "No estás autorizado"
        end
    else Usuario Autorizado
        rect rgba(0, 255, 0, 0.1)
            Agent->>Backend: Notifica autorización exitosa
            Teams-->>Dev: "Eres un usuario permitido"
        end
    end

```
</details>

<details>
<summary><b>Fase 2: Diseño y Validación de Módulos</b></summary>
    
```mermaid
sequenceDiagram
    autonumber
    
    actor Dev as Usuario Solicitante 
    participant Teams as Canal Teams
    participant Backend as Bot Backend
    participant Agent as AI Foundry Agent
    participant MCP as MCP Server (Tools)
    participant Jira as JIRA ITSM
    participant TFC as Terraform Cloud

    rect rgb(255, 255, 204)
        loop Recolección de Requisitos
            Dev->>Teams: Describe necesidad o características del recurso
            Teams->>Agent: Enruta respuesta
            Agent->>Backend: Hace preguntas y recomendaciones
            Backend->>Teams: Muestra sugerencias/preguntas
        end
    end
    
    rect rgb(240, 248, 255)
        Agent->>MCP: ¿Existen módulos privados para esta necesidad?
        MCP->>TFC: Consulta Registro de Módulos Privados
        TFC-->>MCP: Retorna lista de módulos / No match
    end

    alt Módulos NO Cubren Necesidad
        rect rgb(255, 235, 235)
            MCP-->>Agent: No se encontraron módulos compatibles
            Agent->>Backend: Genera notificación de "No Soportado"
            Backend->>Teams: Muestra Tarjeta de Sugerencia Manual
            Teams-->>Dev: "No soportamos este aprovisionamiento. Sugerimos proceso manual."
            Note over Dev, Agent: Fin de la atención (Cierre de sesión)
        end
    else Módulos Cubren Necesidad
        rect rgb(235, 255, 235)
            MCP-->>Agent: Módulos encontrados (ej. storage, keyvault)
            Agent->>MCP: Crea Work Order en JIRA
            MCP->>Jira: Registra Work Order para la trazabilidad.
            Agent->>Agent: Genera código Terraform en base a modulos privados.
            Agent->>Backend: Notifica Orquestación de Módulos (main.tf)
            Backend->>Teams: Notifica el main.tf generado.
            Teams-->>Dev: "Se orquestarán los módulos X, Y para tu necesidad"
        end
    end
```

</details>

<details>
<summary><b>Fase 3: GitOps, Sentinel y FinOps</b></summary>

```mermaid
sequenceDiagram
    autonumber
    
    actor Dev as Usuario Solicitante 
    participant Teams as Canal Teams
    participant Backend as Bot Backend
    participant Agent as AI Foundry Agent
    participant MCP as MCP Server (Tools)
    participant GH as GitHub
    participant TFC as Terraform Cloud

    rect rgb(240, 248, 255)
        Agent->>MCP: Tool: Git Operations (Commit & PR)
        MCP->>GH: Commit a rama 'provisioning'
        MCP->>GH: Create PR hacia rama de ambiente
        Note over GH, TFC: Evento: PR Aperturado
        GH-->>TFC: Webhook: Gatilla Speculative Plan
        Note right of TFC: TFC ejecuta automáticamente:<br/>1. Speculative Plan<br/>2. Sentinel Checks
    end

    rect rgb(255, 255, 204)
        Agent->>MCP: Tool: Interprete (Consultar TFC)
        MCP->>TFC: Obtiene JSON de Plan y Sentinel
        MCP->>MCP: Función 'Interprete': Traduce Plan a Lenguaje Natural
        Agent->>MCP: Tool: FinOps (Calculadora/Azure)
        MCP-->>Agent: Datos de Costos Enriquecidos
    end

    alt Sentinel Fallo HARD
        rect rgb(255, 230, 230)
            Agent->>Backend: Notifica Rechazo Inmediato (Hard)
            Teams-->>Dev: "Rechazado: Política Crítica violada. Detalle: ..."
        end
    else Sentinel Fallo SOFT / OK
        rect rgb(255, 245, 200)
            Agent->>Backend: Envía Plan Interpretado + Sentinel Status + Costos
            Teams-->>Dev: Muestra detalle. ¿Deseas proceder?
        end
    end
```

</details>

<details>
<summary><b>Fase 4: Gobernanza y Aprovisionamiento Final</b></summary>
``` mermaid
sequenceDiagram
    autonumber
    
    actor Dev as Usuario Solicitante 
    actor PO as Product Owner (PO)
    actor Revisor as Revisor Par 
    participant Teams as Canal Teams
    participant Backend as Bot Backend
    participant Agent as AI Foundry Agent
    participant MCP as MCP Server (Tools)
    participant GH as GitHub
    participant TFC as Terraform Cloud
    participant Jira as JIRA ITSM

    Dev->>Teams: Acepta cambios (y solicita excepción si es Soft)
    
    rect rgb(230, 230, 250)
        Note over Dev, Revisor: Proceso de Peer Review
        Dev->>Revisor: Solicita revisión de código
        Revisor-->>Dev: Aprobación OK
        
        Note over Agent, PO: Conformidad del PO (Datos CVT previos)
        Agent->>Backend: Envía solicitud al PO
        Backend->>Teams: Envía Tarjeta Adaptativa al PO
        Teams-->>PO: Solicita Conformidad (Cambios, Sentinel OK, Costos)
        PO->>Teams: Da conformidad
        Teams->>Agent: Enruta aprobación
    end

    Agent->>MCP: Tool: Merge PR
    MCP->>GH: Merge PR a rama destino
    Note over GH, TFC: Comportamiento Natural: Gatilla RUN Final
    GH-->>TFC: Gatilla Run (Plan/Sentinel OK)
    Note right of TFC: TFC en espera de 'Apply' manual
    
    Agent->>MCP: Tool: Terraform Apply (API Call)
    MCP->>TFC: POST /runs/:id/actions/apply
    Agent->>Backend: Notifica "Ejecutando... esto tomará unos minutos"
    Teams-->>Dev: "Aplicando cambios en la nube..."

    alt Apply Exitoso
        rect rgb(204, 255, 204)
            TFC-->>MCP: Apply Finished (Outputs)
            Agent->>Backend: Notifica éxito + Detalle recursos (Name, RG, Subscription)
            Agent->>Backend: Sugiere features adicionales del módulo
            Teams-->>Dev: "Recursos creados con éxito."
            Agent->>MCP: Cierra Work Order en JIRA
            MCP->>Jira: WO Cerrado con trazabilidad total
            MCP-->>Agent: Confirmación
            Agent->>Backend: Genera mensaje de éxito
            Backend->>Teams: Notifica éxito
            Teams-->>Dev: "Aprovisionamiento terminado con éxito"
            Note right of Teams: Mostrar Detalle de los cambios aplicados.
        end
    else Error en el Apply
        rect rgb(255, 220, 220)
            TFC-->>MCP: Error en ejecución
            Agent->>Backend: Notifica error al usuario
            Agent->>MCP: Deriva WO a Soporte N2 (Expert Team)
            MCP->>Jira: WO escalado a Soporte Cloud Governance
            Teams-->>Dev: "Error en el apply. Se ha derivado tu caso a N2."
        end
    end
    
```


