``` typescript
// entrypoint/index.ts
import { randomUUID } from "node:crypto";
import { logger } from "./WinstonLogger.ts";

async function main(_event: unknown, _context: unknown) {
    try {
        contextWrapper.executionId = randomUUID();
        logger.createStream();

        logger.info({
            event: "main",
            details: "Processing event"
        });

        // execução dos serviços
    } catch (error) {
        logger.error({
            event: "main",
            details: error instanceof Error ? error.message : String(error),
        });
    } finally {
        logger.finishStream();
    }
}
```
