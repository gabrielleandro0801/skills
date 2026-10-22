---
name: nodejs-datadog-tracing
description: 'Guia para implementar tracing com Datadog no código em TypeScript.'
---

# Recomendações

- Não presuma nem escolha NADA, siga completamente as recomendações e padrões estabelecidos para tracing com Datadog no projeto.
- Utilize a biblioteca oficial do Datadog (dd-trace) para implementar tracing no seu projeto.
- Garanta que todos os spans importantes sejam criados e finalizados corretamente para capturar o fluxo completo da execução.
- Utilize contextos apropriados para propagar informações de tracing entre diferentes partes do código.
- Evite criar spans desnecessários que possam poluir os dados de tracing.
- Sempre capture e registre erros dentro dos spans para facilitar a análise de falhas.
- Utilize um arquivo Tracer.ts conforme o exemplo abaixo:
``` typescript
import ddTrace, { Span, Tracer } from "dd-trace";
import { logger } from "./logger";

const env: string = process.env.ENV || "";

const SERVICE_NAME: string = "my-app";

let tracer: Tracer;

export const initTracer = () => {
    try {
        if (!["hml", "prd"].includes(env)) {
            return;
        }

        tracer = ddTrace.init({
            plugins: true,
            logger: logger.getLogger(),
            logInjection: true,
            service: SERVICE_NAME,
        });
    } catch (error) {
        logger.error({
            event: "initTracer",
            error: error instanceof Error ? error.message : String(error),
        });
    }
};

export const tagMultiple = (object: { [key: string]: unknown }) => {
    try {
        if (tracer && object) {
            const span: Span | null = tracer.scope().active();
            if (span !== null) {
                span.addTags(object);
            }
        }
    } catch (error) {
        logger.error({
            event: "tagMultiple",
            error: error instanceof Error ? error.message : String(error),
        });
    }
};

export const getSpanIdAndTraceId = () => {
    let trace_id: string = "";
    let span_id: string = "";

    try {
        if (tracer) {
            const span: Span | null = tracer.scope().active();
            if (span !== null) {
                trace_id = span.context().toTraceId();
                span_id = span.context().toSpanId();
            }
        }
    } catch (error) {
        logger.error({
            event: "getSpanIdAndTraceId",
            error: error instanceof Error ? error.message : String(error),
        });
    }

    return {
        trace_id,
        span_id,
    };
};

export const TraceClass = (name: string) => {
    return (clazz) => {
        try {
            if (tracer) {
                Object.getOwnPropertyNames(clazz.prototype)
                    .filter((x) => x !== "constructor")
                    .forEach(
                        (method) =>
                            (clazz.prototype[method] = tracer.wrap(`${name}.${method}`, clazz.prototype[method])),
                    );
            }
        } catch (error) {
            logger.error({
                event: "TraceClass",
                error: error instanceof Error ? error.message : String(error),
            });
        }
    };
};
```
- Sempre peça ao usuário o nome do serviço a ser colocado na constante.
- Importe e utilize a função `initTracer` para inicializar o tracer no início da aplicação, sendo o primeiro import da aplicação, antes dos outros imports da aplicação, garantindo que todas as operações subsequentes sejam rastreadas corretamente.
- Utilize a função `tagMultiple` para adicionar múltiplas tags ao span ativo, garantindo que informações relevantes sejam associadas ao rastreamento.
- Utilize a função `getSpanIdAndTraceId` para obter os IDs de trace e span do span ativo, permitindo correlacionar logs e rastreamentos de forma precisa.
- Utilize o decorator `TraceClass` para automaticamente envolver os métodos de uma classe com o tracer, facilitando o rastreamento de chamadas de métodos sem modificar manualmente cada um deles, conforme o exemplo abaixo:

``` typescript
@TraceClass("MyService")
class MyService {
    myMethod(input: { [key: string]: unknown }) {
        tagMultiple(input);
    }
}
```
- O uso do `tagMultiple` é para acontecer somente nos métodos mais importantes (nos casos de uso, serviços, controladores, etc), onde informações adicionais sobre o rastreamento são relevantes.
- Utilize o `getSpanIdAndTraceId` para obter os IDs de trace e span do span ativo no logging da aplicação, permitindo correlacionar logs com rastreamentos de forma precisa.
