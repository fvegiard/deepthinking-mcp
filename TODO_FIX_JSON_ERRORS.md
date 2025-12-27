# 🔧 DEEPTHINKING-MCP - FIXES CRITIQUES

## 🎯 PROBLÈMES IDENTIFIÉS

**1. Dynamic require('os') - BLOQUANT PERSISTENCE**
- Fichier: `src/utils/file-lock.ts:94`
- Erreur: `Dynamic require of "os" is not supported`
- Impact: Sessions ne persistent pas, crash à l'écriture

**2. Logger pollue stdout - CASSE JSON MCP**
- Fichier: `src/utils/logger.ts:108-120`
- Erreur: `console.log()` va vers stdout au lieu stderr
- Impact: Client MCP reçoit JSON fragmenté, parse errors en cascade

---

## ✅ TODO - PHASE 1 (15min)

### FIX 1: Dynamic require → ESM import
**Fichier:** `src/utils/file-lock.ts`

```typescript
// AVANT (ligne 15, ajouter):
import { promises as fs } from 'fs';
import * as path from 'path';
import { createLogger, LogLevel } from './logger.js';
// AJOUTER:
import os from 'os';

// AVANT (ligne 94):
hostname: require('os').hostname(),

// APRÈS (ligne 94):
hostname: os.hostname(),
```

**Commande:**
```bash
cd D:\GitHub\deepthinking-mcp
# Ouvrir file-lock.ts et appliquer changements ci-dessus
```

---

### FIX 2: Logger stdout → stderr
**Fichier:** `src/utils/logger.ts`

```typescript
// AVANT (lignes 108-120):
switch (entry.level) {
  case LogLevel.DEBUG:
  case LogLevel.INFO:
    console.log(message);  // ❌ STDOUT
    break;
  case LogLevel.WARN:
    console.warn(message);
    break;
  case LogLevel.ERROR:
    console.error(message);
    break;
}

// APRÈS (lignes 108-120):
switch (entry.level) {
  case LogLevel.DEBUG:
  case LogLevel.INFO:
    console.error(message);  // ✅ STDERR
    break;
  case LogLevel.WARN:
    console.error(message);  // ✅ STDERR
    break;
  case LogLevel.ERROR:
    console.error(message);  // ✅ STDERR
    break;
}
```

**Rationale:** MCP utilise stdout pour JSON-RPC, tout log doit aller stderr

---

### FIX 3: Rebuild
```bash
cd D:\GitHub\deepthinking-mcp
npm run build
```

**Vérifier:** 
- `dist/index.js` size ~similaire
- Pas d'erreurs build

---

## ✅ TODO - PHASE 2 (5min)

### TEST 1: Restart Claude Desktop
```bash
# Fermer complètement Claude Desktop
# Rouvrir
```

### TEST 2: Appel simple
```
User → Claude: "Use deepthinking_core with mode deductive"
```

**Vérifier logs:**
```bash
Get-Content C:\Users\fvegi\AppData\Roaming\Claude\logs\mcp-server-deepthinking.log -Tail 50
```

**Critères succès:**
- ❌ Aucune ligne `Unexpected token`, `Context: {`, parsing error
- ✅ Réponse JSON valide
- ✅ Session complète

### TEST 3: Persistence fichiers
```bash
ls C:\Users\fvegi\.claude\deepthinking-sessions\
```

**Critère succès:**
- ✅ Fichiers `.json` créés
- ❌ Aucune erreur "Dynamic require"

---

## ✅ TODO - PHASE 3 (optionnel)

### Optimisation logging
**Fichier:** `claude_desktop_config.json`

```json
{
  "mcpServers": {
    "deepthinking": {
      "command": "node",
      "args": ["D:/GitHub/deepthinking-mcp/dist/index.js"],
      "env": {
        "MCP_LOG_LEVEL": "warn"  // Réduit verbosité
      }
    }
  }
}
```

---

## 📊 RÉSUMÉ ACTIONS

1. ✏️ Edit `src/utils/file-lock.ts` → Import os, ligne 94
2. ✏️ Edit `src/utils/logger.ts` → console.error partout
3. 🔨 `npm run build`
4. 🔄 Restart Claude Desktop
5. ✅ Test appel + vérif logs + persistence

**Temps estimé:** 20min total

---

## 🎯 CRITÈRES VALIDATION

- [ ] Build sans erreurs
- [ ] Logs deepthinking: 0 JSON parsing errors
- [ ] Appel deepthinking_core réussit
- [ ] Fichiers .json dans sessions/
- [ ] Export session fonctionne

---

**Status:** TODO  
**Priorité:** CRITIQUE  
**Bloquant:** Persistence + JSON pollution
