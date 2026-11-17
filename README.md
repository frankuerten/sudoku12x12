# sudoku12x12
O algoritmo verifica a unicidade da solução de um sudoku 12x12:

O algoritmo recebe um sudoku 12x12 e retorna se o sudoku possui zero, uma ou mais de uma solução. Antes de rodar o algoritmo, o usuário deve definir o tamanho das sub-regiões do sudoku, que podem ser de tamanho 3x4, 4x3, 2x6 ou 6x2.

```python
import copy

def imprimir_sudoku(tabuleiro, box_linhas, box_colunas):
    N = 12

    largura = 3
    separador_vertical = "|"
    tamanho_linha = N * largura + (N // box_colunas - 1) * 2

    for i, linha in enumerate(tabuleiro):

        # Separador horizontal entre sub-regiões
        if i > 0 and i % box_linhas == 0:
            print("-" * tamanho_linha)

        for j, valor in enumerate(linha):

            # Separador vertical entre sub-regiões
            if j > 0 and j % box_colunas == 0:
                print(separador_vertical, end=" ")

            # Cada célula ocupa três posições
            print(f"{valor:2} ", end="")

        print()


def solucoes_sudoku(tabuleiro, box_linhas, box_colunas):
    N = 12

    def eh_valido(tab, num, pos):
        row, col = pos

        # Verifica linha
        if num in tab[row]:
            return False

        # Verifica coluna
        for i in range(N):
            if tab[i][col] == num:
                return False

        # Verifica sub-região
        inicio_linha = (row // box_linhas) * box_linhas
        inicio_coluna = (col // box_colunas) * box_colunas

        for i in range(inicio_linha, inicio_linha + box_linhas):
            for j in range(inicio_coluna, inicio_coluna + box_colunas):
                if tab[i][j] == num:
                    return False

        return True

    # Heurística MRV
    def encontrar_melhor_vazio(tab):
        melhor_posicao = None
        melhores_candidatos = None
        menor_quantidade = N + 1

        for i in range(N):
            for j in range(N):

                if tab[i][j] == 0:

                    candidatos = [
                        num for num in range(1, N + 1)
                        if eh_valido(tab, num, (i, j))
                    ]

                    # Nenhum candidato disponível
                    if not candidatos:
                        return (i, j), []

                    # Escolhe a célula com menos candidatos
                    if len(candidatos) < menor_quantidade:
                        menor_quantidade = len(candidatos)
                        melhor_posicao = (i, j)
                        melhores_candidatos = candidatos

                        if menor_quantidade == 1:
                            return melhor_posicao, melhores_candidatos

        return melhor_posicao, melhores_candidatos

    def backtracking(tab, contador_solucoes):

        # Para após encontrar duas soluções
        if contador_solucoes[0] >= 2:
            return

        # Seleciona a célula com menos candidatos
        pos_vazio, candidatos = encontrar_melhor_vazio(tab)

        # Todas as células foram preenchidas
        if pos_vazio is None:
            contador_solucoes[0] += 1
            return

        # Não existe possibilidade de preenchimento
        if not candidatos:
            return

        row, col = pos_vazio

        for num in candidatos:

            tab[row][col] = num

            backtracking(tab, contador_solucoes)

            # Desfaz a tentativa
            tab[row][col] = 0

            if contador_solucoes[0] >= 2:
                return

    tabuleiro_copia = copy.deepcopy(tabuleiro)
    contador = [0]

    backtracking(tabuleiro_copia, contador)

    return contador[0]


def verificar_sudoku(tabuleiro, box_linhas, box_colunas):

    N = 12

    configuracoes = [(3, 4), (4, 3), (2, 6), (6, 2)]

    # Verifica configuração das sub-regiões
    if (box_linhas, box_colunas) not in configuracoes:
        return "Configuração de sub-região inválida"

    def validar_tabuleiro_inicial(tab):

        # Verifica dimensões
        if len(tab) != N:
            return False

        if any(len(linha) != N for linha in tab):
            return False

        # Verifica números permitidos
        for i in range(N):
            for j in range(N):

                valor = tab[i][j]

                if not isinstance(valor, int) or isinstance(valor, bool):
                    return False

                if valor < 0 or valor > N:
                    return False

        # Verifica linhas
        for i in range(N):
            linha = [x for x in tab[i] if x != 0]

            if len(linha) != len(set(linha)):
                return False

        # Verifica colunas
        for j in range(N):
            coluna = [
                tab[i][j]
                for i in range(N)
                if tab[i][j] != 0
            ]

            if len(coluna) != len(set(coluna)):
                return False

        # Verifica sub-regiões
        for box_i in range(0, N, box_linhas):
            for box_j in range(0, N, box_colunas):

                regiao = []

                for i in range(box_i, box_i + box_linhas):
                    for j in range(box_j, box_j + box_colunas):

                        valor = tab[i][j]

                        if valor != 0:
                            regiao.append(valor)

                if len(regiao) != len(set(regiao)):
                    return False

        return True

    # Valida o tabuleiro inicial
    if not validar_tabuleiro_inicial(tabuleiro):
        return "0 soluções"

    # Conta as soluções
    num_solucoes = solucoes_sudoku(
        tabuleiro,
        box_linhas,
        box_colunas
    )

    if num_solucoes == 0:
        return "0 soluções"

    elif num_solucoes == 1:
        return "1 solução"

    else:
        return "mais de uma solução"


if __name__ == "__main__":

    sudoku_modelo = [
    [ 1,  0,  3,  0,  5,  0,  7,  0,  9,  0, 11,  0],
    [ 5,  6,  0,  8,  9, 10,  0, 12,  1,  2,  0,  4],
    [ 9, 10, 11,  0,  1,  2,  3,  0,  5,  6,  7,  8],

    [ 2,  3,  4,  5,  0,  7,  8,  9,  0, 11, 12,  1],
    [ 6,  0,  8,  9, 10, 11,  0,  1,  2,  3,  4,  5],
    [10, 11, 12,  1,  2,  3,  4,  5,  6,  0,  8,  9],

    [ 3,  4,  5,  6,  7,  8,  9,  0, 11, 12,  1,  2],
    [ 7,  8,  9, 10, 11, 12,  1,  2,  3,  4,  5,  6],
    [11, 12,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10],

    [ 4,  5,  6,  7,  8,  9, 10, 11, 12,  1,  2,  3],
    [ 8,  9, 10, 11, 12,  1,  2,  3,  4,  5,  6,  7],
    [12,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11]
]
        

    # Escolha da configuração das sub-regiões
    box_linhas = 3
    box_colunas = 4

    print("\nSudoku inicial:\n")
    imprimir_sudoku(sudoku_modelo, box_linhas, box_colunas)

    resultado = verificar_sudoku(
        sudoku_modelo,
        box_linhas,
        box_colunas
    )


```


Para utilizar o algoritmo, basta colá-lo no Google Colab, editar as células do sudoku_modelo e definir o tamanho das sub-regiões em box_linhas e box_colunas. Note que os zeros representam os espaços vazios. Utilizando o exemplo dado acima, o programa retorna "Quantidade de soluções: 1 solução."

print("\nQuantidade de soluções:", resultado)
