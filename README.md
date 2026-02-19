# diagram

```mermaid
sequenceDiagram
    autonumber
    
    actor Dev as Usuario Solicitante 
    actor PO as Product Owner (PO)
    actor Par as Revisor Par 
    participant Teams as Canal Teams
    participant Bot as Azure Bot Service
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
    Dev->>Teams: Responde app
    Teams->>Bot: Pasa respuesta
    Bot->>Backend: Enruta
    Backend->>Agent: Texto con la app
    
    rect rgb(240, 248, 255)
        Note right of Agent: Validaciones de Autorización Vía MCP
        Agent->>MCP: Solicita validaciones
        MCP->>CVT: Valida si app existe
        MCP->>AAD: Valida si pertenece a Grupo de Red
        MCP->>TFC: Valida si pertenece a Team en Terraform Cloud
        MCP-->>Agent: Retorna resultados de validación
    end

    alt Usuario NO Autorizado
        Agent->>Backend: Genera mensaje de rechazo
        Backend->>Teams: Notifica falta de permisos
        Teams-->>Dev: "No estás autorizado para esta app"
    else Usuario Autorizado
        Agent->>Backend: Genera mensaje de éxito de autorización
        Backend->>Teams: Notifica autorización exitosa
        Teams-->>Dev: "Eres un usuario permitido. ¿Qué necesitas aprovisionar?"
        
        %% Fase 2: Entendimiento de la Necesidad (Conversacional)
        loop Recolección de Requisitos
            Dev->>Teams: Describe características del recurso
            Teams->>Agent: Enruta respuesta
            Agent->>Backend: Hace preguntas y recomendaciones
            Backend->>Teams: Muestra sugerencias/preguntas
        end
        
        %% Fase 3: Módulos, Trazabilidad y Código
        Agent->>MCP: Consulta módulos privados en TFC
        MCP-->>Agent: Módulos cubren la necesidad
        Agent->>MCP: Crea Work Order en JIRA
        MCP->>Jira: Registra Work Order (Trazabilidad)
        Agent->>Agent: Genera código Terraform
        Agent->>Backend: Genera notificación de Módulos
        Backend->>Teams: Notifica al usuario
        Teams-->>Dev: "Se orquestarán los módulos X, Y, Z para tu necesidad"
        
        %% Fase 4: PR y Plan
        Agent->>MCP: Realiza Pull Request en GitHub
        MCP->>GH: Crea PR
        Agent->>Backend: Genera notificación de PR
        Backend->>Teams: Notifica al usuario
        Teams-->>Dev: "Pull Request creado exitosamente"
        Dev-->>Teams: "holi"
        MCP->>GH: Realiza Merge del PR
        GH->>TFC: Gatilla Auto Run (Genera Plan)
        TFC-->>MCP: Retorna Plan + Sentinel Status + Costos
        MCP-->>Agent: JSON estructurado
        
        %% Fase 5: Revisión del Solicitante y Par
        Agent->>Backend: Envía detalles del Plan
        Backend->>Teams: Muestra Tarjeta Adaptativa (Plan)
        Teams-->>Dev: Decide si aplica el cambio
        Dev->>Teams: Aprueba el plan (intención)
        
        Dev->>Par: Solicita revisión de código (Peer Review)
        Par-->>Dev: Revisión OK
        
        %% Fase 6: Políticas Sentinel y Costos
        alt Políticas Sentinel Falla / Con Observaciones
            Agent->>Backend: Genera excepción
            Backend->>Teams: Muestra Tarjeta de Rechazo/Regularización
            Teams-->>Dev: Aprovisionamiento pausado (Observaciones Sentinel)
        else Sentinel OK (Sin Observaciones)
            Agent->>Backend: Prepara Tarjeta de Costos e Info
            Backend->>Teams: Muestra costo de infra y validaciones OK
            Teams-->>Dev: Revisa costos
            
            %% Fase 7: Conformidad del Product Owner
            Backend->>Teams: Envía Tarjeta Adaptativa al PO
            Teams-->>PO: Solicita Conformidad (Cambios, Sentinel OK, Costos)
            PO->>Teams: Da conformidad
            Teams->>Agent: Enruta aprobación
            
            %% Fase 8: Apply y Cierre
            Agent->>MCP: Solicita ejecución de Apply
            MCP->>TFC: Terraform Apply
            
            alt Aprovisionamiento Exitoso
                TFC-->>MCP: Apply finalizado sin errores
                MCP->>Jira: Cierra Work Order
                MCP-->>Agent: Confirmación
                Agent->>Backend: Genera mensaje de éxito
                Backend->>Teams: Notifica éxito
                Teams-->>Dev: "Aprovisionamiento terminado con éxito"
            else Error en Aprovisionamiento
                TFC-->>MCP: Error en el Apply
                MCP->>Jira: Deriva Work Order a Soporte Cloud Governance
                MCP-->>Agent: Detalles del error
                Agent->>Backend: Genera mensaje de derivación
                Backend->>Teams: Notifica error
                Teams-->>Dev: "Error. Ticket derivado a soporte para revisión"
            end
        end
    end
```
