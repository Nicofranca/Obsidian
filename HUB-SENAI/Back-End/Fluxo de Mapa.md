### 1. Layout da Sala

A API deve tratar cada sala como um "Grid" (plano com eixos X e Y).


- **Tabela `mapa_sala`**:
    
    - `id`: 

    - `id_sala`: - vem do token
        
    - `nome`: Lab de Informática - 204
        
    - `dimensao_x`: 8 (quadrados de largura)
        
    - `dimensao_y`: 3 (quadrados de comprimento)
        
- **Tabela `Assento`**:
    
    - `id`: 
        
    - `sala_id`: sala-211
        
    - `posicao_x`: 2 (coluna 2 do grid)
        
    - `posicao_y`: 4 (linha 4 do grid)
        
    - /`label`: SN322 - numero senai
        
    - /`tipo`: mesa_normal, computador, cadeirante
        


### 2. Alocação 


- **Tabela `Mapa_Ocupacao`**:
    
    - `id`: 
        
    - `turma_id`: "turma-1" - do token
        
    - `sala_id`: "sala-211"
        
    - `assento_id`: "assento-01"
        
    - `aluno_id`: "user-123" - token













[[HUB-SENAI]]