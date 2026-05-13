### 1. O Modelo do Espaço Físico (Layout da Sala)

Em vez de engessar o sistema, a API deve tratar cada sala como um "Grid" (um plano com eixos X e Y).

No banco de dados desta _feature_, você teria algo assim:

- **Tabela `Sala`**:
    
    - `id`: "sala-211"
        
    - `nome`: "Laboratório de Informática 1"
        
    - `dimensao_x`: 10 (quantos "quadrados" de largura a sala tem)
        
    - `dimensao_y`: 10 (quantos "quadrados" de comprimento)
        
- **Tabela `Assento`**:
    
    - `id`: "assento-01"
        
    - `sala_id`: "sala-211"
        
    - `posicao_x`: 2 (coluna 2 do grid)
        
    - `posicao_y`: 4 (linha 4 do grid)
        
    - `label`: "PC-01" ou "A1" (a numeração padrão que o aluno vai enxergar)
        
    - `tipo`: "mesa_normal", "computador", "cadeirante" (útil para acessibilidade).
        

**Por que fazer assim?** Porque as salas do SENAI têm formatos diferentes. Algumas são em "U", outras em fileiras tradicionais, outras em ilhas. O banco guardando as posições X e Y permite que o frontend desenhe exatamente o formato real da sala.

### 2. O Modelo de Alocação (A "Fotografia" da Turma)

Agora entra a lógica de cruzar a Turma (que vem do token) com a Sala física.

Você cria uma tabela de vínculo:

- **Tabela `Mapa_Ocupacao`**:
    
    - `id`: UUID
        
    - `turma_id`: "turma-1" (Você sabe que o professor/aluno tem acesso porque validou no token)
        
    - `sala_id`: "sala-211"
        
    - `assento_id`: "assento-01"
        
    - `aluno_id`: "user-123" (o ID que veio do Core)
        
    - `fixo`: booleano (para saber se o professor travou o aluno naquela cadeira ou se foi alocação automática).












[[HUB-SENAI]]