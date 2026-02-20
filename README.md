# diagram
<details>
<summary><b>Fase 1: Validación de Identidad y Autorización</b></summary>

```mermaid

sequenceDiagram
    autonumber
    
    actor Dev as Usuario Solicitante 
    actor PO as Product Owner (PO)
    actor Revisor as Revisor Par 
    participant Teams as Canal Teams
    participant Bot as Azure Bot Service (Enrutador)
    participant Backend as Bot Backend
    participant Agent as AI Foundry Agent
    participant MCP as MCP Server (Tools)
    participant AAD as Azure AD
    participant CVT as Portal App CVT
    participant Jira as JIRA ITSM
    participant GH as GitHub
    participant TFC as Terraform Cloud

    %% Fase 1: Intención y Validación de Identidad Extensiva
    Dev->>Teams: Solicita aprovisionar recurso Cloud
    Teams->>Bot: Envía mensaje
    Bot->>Backend: Enruta payload
    Backend->>Agent: Envía texto (Sesión propagada)
    Agent->>Backend: Pregunta aplicación impactada
    Backend->>Teams: Renderiza Tarjeta Adaptativa
    Teams-->>Dev: ¿Cuál es la app impactada?
    Dev->>Teams: Responde app (cod 4 digitos)
    Teams->>Bot: Pasa respuesta
    Bot->>Backend: Enruta
    Backend->>Agent: Texto con la app
    
    rect rgba(0, 120, 215, 0.1)
        Note right of Agent: Validaciones de Autorización Vía MCP
        Agent->>MCP: Solicita validaciones
        Note right of MCP: La Tool consulta CVT una sola vez
        MCP->>CVT: GET /apps/YAPE
        CVT-->>MCP: {exists: true, owner: "po_user@empresa.com", status: "Vigente", nombreApp: "YAPE"}
        MCP->>AAD: Valida ID y Grupos de Red "VSDP_USER_PROD" o "VSDP_ADMIN_PROD"
        MCP->>TFC: Valida si pertenece a Team "HTFC-CONSUMER-PROJECT-YAPE" en Terraform Cloud
        MCP-->>Agent: Retorna JSON Enriquecido (Data CVT + Data AAD + Data TFC)
    end
    
    Note over Agent: El Agente guarda al PO en su contexto

    alt Usuario NO Autorizado
        rect rgba(255, 0, 0, 0.1)
            Agent->>Backend: Genera mensaje de rechazo
            Backend->>Teams: Notifica falta de permisos
            Teams-->>Dev: "No estás autorizado para esta app"
        end
    else Usuario Autorizado
        rect rgba(0, 255, 0, 0.1)
            Agent->>Backend: Genera mensaje de éxito de autorización
            Backend->>Teams: Notifica autorización exitosa
            Teams-->>Dev: "Eres un usuario permitido. ¿Qué necesitas aprovisionar?"
        end
    end
```
</details>

<details>
<summary><b>Fase 2 y 3: Entendimiento y Validación de Módulos</b></summary>
    
```mermaid
sequenceDiagram
    autonumber
    
    actor Dev as Usuario Solicitante 
    participant Teams as Canal Teams
    participant Backend as Bot Backend
    participant Agent as AI Foundry Agent
    participant MCP as MCP Server (Tools)
    participant TFC as Terraform Cloud
    participant Jira as JIRA ITSM

    %% Fase 2: Entendimiento de la Necesidad (Conversacional)
    rect rgba(255, 255, 0, 0.1)
        loop Recolección de Requisitos
            Dev->>Teams: Describe necesidad o características del recurso
            Teams->>Agent: Enruta respuesta
            Agent->>Backend: Hace preguntas y recomendaciones
            Backend->>Teams: Muestra sugerencias/preguntas
        end
    end
    
    %% Fase 3: Validación de Módulos (LA NUEVA ITERACIÓN)
    rect rgba(0, 120, 215, 0.1)
        Agent->>MCP: ¿Existen módulos privados para esta necesidad?
        MCP->>TFC: Consulta Registro de Módulos Privados
        TFC-->>MCP: Retorna lista de módulos / No match
    end

    alt Módulos NO Cubren Necesidad
        rect rgba(255, 0, 0, 0.1)
            MCP-->>Agent: No se encontraron módulos compatibles
            Agent->>Backend: Genera notificación de "No Soportado"
            Backend->>Teams: Muestra Tarjeta de Sugerencia Manual
            Teams-->>Dev: "No soportamos este aprovisionamiento. Sugerimos proceso manual."
            Note over Dev, Agent: Fin de la atención (Cierre de sesión)
        end
    else Módulos Cubren Necesidad
        rect rgba(0, 255, 0, 0.1)
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
<summary><b>Fase 4 y 5: GitOps, Interpretación y FinOps</b></summary>

```mermaid
sequenceDiagram
    autonumber
    
    participant Agent as AI Foundry Agent
    participant MCP as MCP Server (Tools)
    participant GH as GitHub
    participant TFC as Terraform Cloud

    %% Fase 4: GitOps y Comportamiento Natural de TFC
    rect rgba(0, 120, 215, 0.1)
        Agent->>MCP: Tool: Git Operations (Commit & PR)
        MCP->>GH: Commit a rama 'provisioning'
        MCP->>GH: Create PR hacia rama de ambiente
        Note over GH, TFC: Evento: PR Aperturado
        GH-->>TFC: Webhook: Gatilla Speculative Plan
        Note right of TFC: TFC ejecuta automáticamente:<br/>1. Speculative Plan<br/>2. Sentinel Checks
    end

    %% Fase 5: Interpretación y FinOps
    rect rgba(255, 255, 0, 0.1)
        Agent->>MCP: Tool: Interprete (Consultar TFC)
        MCP->>TFC: Obtiene JSON de Plan y Sentinel
        MCP->>MCP: Función 'Interprete': Traduce Plan a Lenguaje Natural
        Agent->>MCP: Tool: FinOps (Calculadora/Azure)
        MCP-->>Agent: Datos de Costos Enriquecidos
    end
```

</details>



<summary><b>Fase 4 y 5: GitOps, Interpretación y FinOps</b></summary>
    
```mermaid
sequenceDiagram
    autonumber
    
    participant Agent as AI Foundry Agent
    participant MCP as MCP Server (Tools)
    participant GH as GitHub
    participant TFC as Terraform Cloud

    %% Fase 4: GitOps y Comportamiento Natural de TFC
    rect rgba(0, 120, 215, 0.1)
        Agent->>MCP: Tool: Git Operations (Commit & PR)
        MCP->>GH: Commit a rama 'provisioning'
        MCP->>GH: Create PR hacia rama de ambiente
        Note over GH, TFC: Evento: PR Aperturado
        GH-->>TFC: Webhook: Gatilla Speculative Plan
        Note right of TFC: TFC ejecuta automáticamente:<br/>1. Speculative Plan<br/>2. Sentinel Checks
    end

    %% Fase 5: Interpretación y FinOps
    rect rgba(255, 255, 0, 0.1)
        Agent->>MCP: Tool: Interprete (Consultar TFC)
        MCP->>TFC: Obtiene JSON de Plan y Sentinel
        MCP->>MCP: Función 'Interprete': Traduce Plan a Lenguaje Natural
        Agent->>MCP: Tool: FinOps (Calculadora/Azure)
        MCP-->>Agent: Datos de Costos Enriquecidos
    end
    
```
</details>

<details>
<summary><b>Fase 6, 7 y 8: Gobernanza, Apply y Cierre</b></summary>

```mermaid
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

    %% Fase 6: Lógica de Sentinel y Decisión del Usuario
    alt Sentinel Fallo HARD
        rect rgba(255, 0, 0, 0.1)
            Agent->>Backend: Notifica Rechazo Inmediato (Hard)
            Teams-->>Dev: "Rechazado: Política Crítica violada. Detalle: ..."
        end
    else Sentinel Fallo SOFT / OK
        rect rgba(0, 255, 0, 0.1)
            Agent->>Backend: Envía Plan Interpretado + Sentinel Status + Costos
            Teams-->>Dev: Muestra detalle. ¿Deseas proceder?
        end

        Dev->>Teams: Acepta cambios (y solicita excepción si es Soft)
        
        %% Fase 7: Aprobaciones
        rect rgba(100, 100, 255, 0.1)
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

        %% Fase 8: Merge y Ejecución de Apply
        Agent->>MCP: Tool: Merge PR
        MCP->>GH: Merge PR a rama destino
        Note over GH, TFC: Comportamiento Natural: Gatilla RUN Final
        GH-->>TFC: Gatilla Run (Plan/Sentinel OK)
        Note right of TFC: TFC en espera de 'Apply' manual
        
        Agent->>MCP: Tool: Terraform Apply (API Call)
        MCP->>TFC: POST /runs/:id/actions/apply
        Agent->>Backend: Notifica "Ejecutando aprovisionamiento... esto tomará unos minutos"
        Teams-->>Dev: "Aplicando cambios en la nube..."

        alt Apply Exitoso
            rect rgba(0, 255, 0, 0.1)
                TFC-->>MCP: Apply Finished (Outputs)
                Agent->>Backend: Notifica éxito + Detalle recursos (Name, RG, Subscription)
                Agent->>Backend: Sugiere features adicionales del módulo
                Teams-->>Dev: "Recursos creados con éxito. Puedes habilitar X feature luego."
                Agent->>MCP: Cierra Work Order en JIRA
                MCP->>Jira: WO Cerrado con trazabilidad total
                MCP-->>Agent: Confirmación
                Agent->>Backend: Genera mensaje de éxito
                Backend->>Teams: Notifica éxito
                Teams-->>Dev: "Aprovisionamiento terminado con éxito"
                Note right of Teams: Mostrar Detalle de los cambios aplicados.
            end
        else Error en el Apply
            rect rgba(255, 0, 0, 0.1)
                TFC-->>MCP: Error en ejecución
                Agent->>Backend: Notifica error al usuario
                Agent->>MCP: Deriva WO a Soporte N2 (Expert Team)
                MCP->>Jira: WO escalado a Soporte Cloud Governance
                Teams-->>Dev: "Error en el apply. Se ha derivado tu caso a N2."
            end
        end
    end
```
</details>

