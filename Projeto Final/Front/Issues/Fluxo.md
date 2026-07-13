TASK-01 — Tipos RoomMapViewResponse
Define os tipos TS do payload da API (mapa, grid, alocações, alunos sem lugar). Não tem UI.
{ suggested: true, map: null, grid: {...}, allocations: [...], unassignedStudent: [...] }

TASK-02 — Service mapaDeSalaService
Funções que chamam a API do backend (buscar mapa, criar, salvar alocações).
getRoomMapView(roomId, classId) → GET /api/mapas/salas/{roomId}/turmas/{classId}

TASK-03 — Funções puras de transição
A lógica de "o que acontece quando clico num assento/aluno" — sem React, só funções testáveis.
resolveSeatClick(estado, assento) → novo estado (aluno movido/trocado)

TASK-04 — Hook useMapaDeSala
Junta service + transições num hook 'use client': estado de edição, seleção, salvar.
const { mode, draftAllocations, handleSeatClick, handleSave } = useMapaDeSala(...)

TASK-05 — RoomMapHeader
Barra fina no topo com o "breadcrumb" da tela.
┌─────────────────────────────────┐
│ Sala 113 · MIDS-78 · Mapa de sala│
└─────────────────────────────────┘

TASK-06 — RoomClassSelector
Wizard de 2 passos pra escolher sala e turma antes de entrar no mapa.
[Selecionar Sala] [Selecionar Turma]
 🔍 Buscar...
 204 · Laboratório de informática
 113 · Sala de aula

TASK-07 — Toolbar + Modal + Aviso
3 componentes: botões de ação, modal de confirmação de "limpar mapa", aviso de erro.
[Editar Mapa]   →   [Limpar Mapa] [Salvar Alterações]

  ⚠ confirmação de: Exclusão
  Tem certeza que deseja limpar o mapa?
  [Cancelar]  [Limpar Mapa]

TASK-08 — MapSkeleton
Placeholder cinza animado enquanto o mapa carrega.
▬▬▬▬▬▬▬▬▬▬▬▬ (header)
◯ ▬▬   ◯ ▬▬   ◯ ▬▬   ◯ ▬▬   (assentos)

TASK-09 — MapaDeSalaView
O componente que junta tudo (header + grid + sidebar + toolbar + modal) numa tela só.
┌ RoomMapHeader ──────────────┐
│ [Grid de assentos]  [Sidebar│
│                      alunos]│
│ [Toolbar: Salvar/Limpar]    │
└──────────────────────────────┘

TASK-10 — Páginas do domínio
Server Components que buscam os dados e tratam erro (404, 403, rede).
async function PageMapaDeSala() {
  const data = await getRoomMapView(...)
  return <MapaDeSalaView apiData={data} .../>
}

TASK-11 — Wiring de rotas
Liga tudo às URLs reais em apps/root (/mapa-salas, /mapa-salas/{roomId}/{classId}).
/mapa-salas              → RoomClassSelector
/mapa-salas/113/MIDS-78  → MapaDeSalaView

TASK-12 — MapEmptyState
Tela de "sala sem layout configurado" (raro — problema de infra, não de uso normal).
        [ícone de carteira com "?"]
         Mapa Não Encontrado
  Não há um mapa configurado para esta turma
       [+ Criar Mapa de Sala]
⚠️ achado da auditoria: esse botão hoje não tem pra onde ir (ver TASK-12).

TASK-13 — Fluxo do aluno (somente leitura)
Versão sem edição: aluno escolhe a sala (1 passo) e vê só o próprio lugar destacado.
Encontre seu lugar selecionando a sala...
🔍 [Sala 113]

[Grid de assentos, aluno vê apenas]
     ● ← "Seu lugar está aqui" (ponto azul)
Seu lugar está localizado no ponto azul
Sem sidebar, sem toolbar, sem edição.



207