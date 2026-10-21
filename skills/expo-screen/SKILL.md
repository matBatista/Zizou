---
name: expo-screen
description: Use ao adicionar uma tela, aba ou modal num app Expo Router (LifeOS mobile ou ManagerFC mobile) com estado num store Zustand persistido em AsyncStorage (local-first).
---

# Nova tela/aba no Expo Router + store Zustand persistido

Projetos de referência:
- **LifeOS** `~/Claude/Projects/LIFEOS/mobile` — Expo 56, abas em `app/(tabs)/`, modais `app/add-*.tsx`, stores persistidos em `store/*.ts`, tipos em `lib/db/types.ts`, tema `constants/theme.ts` (`Colors`, `FontSize`).
- **ManagerFC** `~/Claude/Projects/foot_manager/mobile` — Expo 52, abas em `app/(career)/`, a tela de verdade fica em `screens/*Screen.tsx` e a rota só re-exporta; stores em `store/*.ts` **sem** persist (dados vêm da API via `services/*.ts`); tema `constants/theme.ts` (`C`).

## 1. Store (local-first)

Padrão real (`LIFEOS/mobile/store/habits.ts`):

```ts
import { create } from 'zustand'
import { persist, createJSONStorage } from 'zustand/middleware'
import AsyncStorage from '@react-native-async-storage/async-storage'
import { nanoid } from '../lib/utils/nanoid'

interface NotesStore {
  notes: Note[]
  addNote: (n: Omit<Note, 'id' | 'createdAt'>) => void
}

export const useNotesStore = create<NotesStore>()(
  persist(
    (set) => ({
      notes: [],
      addNote: (n) =>
        set((s) => ({ notes: [...s.notes, { ...n, id: nanoid(), createdAt: new Date().toISOString() }] })),
    }),
    { name: 'lifeos-notes', storage: createJSONStorage(() => AsyncStorage) },
  ),
)
```

- `name` único por store (prefixo `lifeos-`). Mudou o shape? adicione `version` + `migrate` no persist — senão dados antigos quebram a tela.
- Tipos novos em `lib/db/types.ts`. Datas como string `YYYY-MM-DD` (`lib/utils/dates.ts::toDateStr`).
- No ManagerFC, AsyncStorage **não está instalado**: `npx expo install @react-native-async-storage/async-storage` antes de usar persist (use `expo install`, não `npm i`, para casar a versão com o SDK).

## 2. Aba

1. Crie `app/(tabs)/notes.tsx` (LifeOS) ou `app/(career)/notes.tsx` + `screens/NotesScreen.tsx` (ManagerFC).
2. Registre no layout do grupo — o arquivo sozinho vira rota, mas título/ícone vêm do `_layout.tsx`:
   - LifeOS `app/(tabs)/_layout.tsx`: `<Tabs.Screen name="notes" options={{ title: 'Notas', tabBarIcon: ({ focused }) => <TabIcon emoji="📝" focused={focused} /> }} />`
   - ManagerFC `app/(career)/_layout.tsx`: adicione em **três** lugares — array `TABS`, `<Tabs.Screen>` do layout mobile e `<Tabs.Screen name>` do `WideLayout` (web/tablet).

## 3. Modal (tela de "adicionar")

Crie `app/add-note.tsx` e registre em `app/_layout.tsx` com `presentation: 'modal'` (copie o bloco de `add-habit`). Navegue com `router.push('/add-note')` e feche com `router.back()`.

## 4. Verificar

```bash
cd ~/Claude/Projects/LIFEOS/mobile && npx tsc --noEmit && npx expo start --web
cd ~/Claude/Projects/foot_manager/mobile && npx tsc --noEmit && npx expo start --web --port 19006
```
Abra a aba, crie um item, recarregue a página: no LifeOS o item tem que continuar lá (persistência).
