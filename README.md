# Instalar a biblioteca DEAP
!pip install deap

# Importar bibliotecas necessárias
import random
from deap import base, creator, tools, algorithms

# Definindo a classe de pontos de entrega
pontos = {
    1: "Memorial Paulo Freire",
    2: "Praça das Flores - UFERSA Angicos, RN",
    3: "Restaurante Universitário",
    4: "Biblioteca Campus Angicos",
    5: "Bloco 1 de Aulas",
    6: "Bloco 2 de Aulas",
    7: "Bloco administrativo - UFERSA Angicos, RN",
    8: "Laboratórios 1 - UFERSA Angicos, RN",
    9: "Laboratórios 2 - UFERSA Angicos, RN",
    10: "Bloco 1 de Professores",
    11: "Bloco 2 de Professores",
    12: "Estacionamento",
    13: "Garagem da UFERSA Angicos",
    14: "Ginásio Poliesportivo da UFERSA Angicos",
    15: "Almoxarifado da UFERSA Angicos"
}

# Lista de pontos para facilitar o acesso
pontos_lista = list(pontos.values())

# Matriz de distâncias predefinidas (em metros)
# Cada linha e coluna corresponde a um ponto na ordem da lista `pontos_lista`
distancias = [
    # Memorial Paulo Freire
    [0, 500, 400, 300, 200, 600, 700, 800, 900, 1000, 1100, 1200, 1300, 1400, 1500],
    # Praça das Flores - UFERSA Angicos, RN
    [500, 0, 350, 450, 550, 650, 750, 850, 950, 1050, 1150, 1250, 1350, 1450, 1550],
    # Restaurante Universitário
    [400, 350, 0, 250, 300, 400, 500, 600, 700, 800, 900, 1000, 1100, 1200, 1300],
    # Biblioteca Campus Angicos
    [300, 450, 250, 0, 100, 200, 300, 400, 500, 600, 700, 800, 900, 1000, 1100],
    # Bloco 1 de Aulas
    [200, 550, 300, 100, 0, 150, 250, 350, 450, 550, 650, 750, 850, 950, 1050],
    # Bloco 2 de Aulas
    [600, 650, 400, 200, 150, 0, 350, 450, 550, 650, 750, 850, 950, 1050, 1150],
    # Bloco administrativo - UFERSA Angicos, RN
    [700, 750, 500, 300, 250, 350, 0, 450, 550, 650, 750, 850, 950, 1050, 1150],
    # Laboratórios 1 - UFERSA Angicos, RN
    [800, 850, 600, 400, 350, 450, 450, 0, 650, 750, 850, 950, 1050, 1150, 1250],
    # Laboratórios 2 - UFERSA Angicos, RN
    [900, 950, 700, 500, 450, 550, 550, 650, 0, 850, 950, 1050, 1150, 1250, 1350],
    # Bloco 1 de Professores
    [1000, 1050, 800, 600, 550, 650, 650, 750, 850, 0, 1050, 1150, 1250, 1350, 1450],
    # Bloco 2 de Professores
    [1100, 1150, 900, 700, 650, 750, 750, 850, 950, 1050, 0, 1250, 1350, 1450, 1550],
    # Estacionamento
    [1200, 1250, 1000, 800, 750, 850, 850, 950, 1050, 1152, 1250, 0, 1350, 1450, 1550],
    # Garagem da UFERSA Angicos
    [1300, 1350, 1100, 900, 850, 950, 950, 1050, 1150, 1250, 1350, 1350, 0, 1450, 1550],
    # Ginásio Poliesportivo da UFERSA Angicos
    [1400, 1450, 1200, 1000, 950, 1050, 1050, 1150, 1250, 1350, 1450, 1450, 1450, 0, 1550],
    # Almoxarifado da UFERSA Angicos
    [1500, 1550, 1300, 1100, 1050, 1150, 1150, 1250, 1350, 1450, 1550, 1550, 1550, 1550, 0]
]

# Função para calcular a distância total de um percurso
def calcular_distancia_total(individuo, distancias, pontos_selecionados):
    total = 0
    # o individuo agora é uma lista de indices, nao o nome do ponto
    # por isso ele usa o index no pontos_selecionados
    for i in range(len(individuo) - 1):
        ponto1_idx = individuo[i]
        ponto2_idx = individuo[i + 1]
        ponto1 = pontos_selecionados[ponto1_idx]
        ponto2 = pontos_selecionados[ponto2_idx]
        total += distancias[list(pontos.values()).index(ponto1)][list(pontos.values()).index(ponto2)]
    # Volta ao ponto inicial
    ponto1_idx = individuo[-1]
    ponto2_idx = individuo[0]
    ponto1 = pontos_selecionados[ponto1_idx]
    ponto2 = pontos_selecionados[ponto2_idx]
    total += distancias[list(pontos.values()).index(ponto1)][list(pontos.values()).index(ponto2)]
    return total

# Função de avaliação do fitness (distância total)
def avaliar(individuo, pontos_selecionados):
    # nao precisa mais do pontos_selecionados_map, pois agora estamos
    # enviando apenas uma lista de indices para o calcular_distancia_total
    distancia_total = calcular_distancia_total(individuo, distancias, pontos_selecionados)
    return (distancia_total,)

# Definição dos tipos do DEAP
if "FitnessMin" in creator.__dict__:
    del creator.FitnessMin
if "Individual" in creator.__dict__:
    del creator.Individual

creator.create("FitnessMin", base.Fitness, weights=(-1.0,))
creator.create("Individual", list, fitness=creator.FitnessMin)

# Função para inicializar a população
def inicializar_populacao(toolbox, tamanho_populacao):
    return toolbox.population(n=tamanho_populacao)

# Função do algoritmo genético
def algoritmo_genetico(distancias, pontos_selecionados, taxa_mutacao_inicial=0.1, elitismo_size=2, max_geracoes_sem_melhoria=10):
    num_pontos = len(pontos_selecionados)

    # Ajustar número de gerações e tamanho da população
    # se o num_pontos for maior que 100, entao usaremos valores menores
    # para que o algoritmo nao demore tanto tempo
    if num_pontos > 100:
        num_geracoes = 10 * num_pontos  # Reduzido para maior eficiência
        tamanho_populacao = 2 * num_pontos  # Reduzido para maior eficiência
    else:
        num_geracoes = 20 * num_pontos  # Reduzido para maior eficiência
        tamanho_populacao = 5 * num_pontos  # Reduzido para maior eficiência

    indices_pontos = list(range(num_pontos)) # Corrected: Use a list of indices
    toolbox = base.Toolbox()
    toolbox.register("indices", random.sample, indices_pontos, len(indices_pontos))
    toolbox.register("individual", tools.initIterate, creator.Individual, toolbox.indices)
    toolbox.register("population", tools.initRepeat, list, toolbox.individual)

    toolbox.register("mate", tools.cxOrdered)
    toolbox.register("mutate", tools.mutShuffleIndexes, indpb=taxa_mutacao_inicial)
    toolbox.register("select", tools.selTournament, tournsize=2)  # Tamanho do torneio reduzido
    toolbox.register("evaluate", avaliar, pontos_selecionados=pontos_selecionados)

    pop = inicializar_populacao(toolbox, tamanho_populacao)
    fitnesses = list(map(toolbox.evaluate, pop))
    for ind, fit in zip(pop, fitnesses):
        ind.fitness.values = fit

    best_individuals = tools.selBest(pop, elitismo_size)
    melhor_fitness = best_individuals[0].fitness.values[0]
    geracoes_sem_melhoria = 0

    for g in range(num_geracoes):
        # Ajustar taxa de mutação dinamicamente
        taxa_mutacao = taxa_mutacao_inicial * (1 - g / num_geracoes)
        toolbox.register("mutate", tools.mutShuffleIndexes, indpb=taxa_mutacao)

        offspring = toolbox.select(pop, len(pop) - elitismo_size)
        offspring = list(map(toolbox.clone, offspring))

        for child1, child2 in zip(offspring[::2], offspring[1::2]):
            if random.random() < 0.7:
                toolbox.mate(child1, child2)
                del child1.fitness.values
                del child2.fitness.values

        for mutant in offspring:
            if random.random() < taxa_mutacao:
                toolbox.mutate(mutant)
                del mutant.fitness.values

        invalid_ind = [ind for ind in offspring if not ind.fitness.valid]
        fitnesses = map(toolbox.evaluate, invalid_ind)
        for ind, fit in zip(invalid_ind, fitnesses):
            ind.fitness.values = fit

        pop[:] = offspring + best_individuals
        best_individuals = tools.selBest(pop, elitismo_size)

        # Verificar convergência
        novo_melhor_fitness = best_individuals[0].fitness.values[0]
        if novo_melhor_fitness < melhor_fitness:
            melhor_fitness = novo_melhor_fitness
            geracoes_sem_melhoria = 0
        else:
            geracoes_sem_melhoria += 1

        # Critério de parada por convergência
        if geracoes_sem_melhoria >= max_geracoes_sem_melhoria:
            print(f"Convergência atingida após {g + 1} gerações.")
            break

    # Corrected: Return the best individual and the list of selected points
    return tools.selBest(pop, 1)[0], pontos_selecionados

# Função para exibir a matriz de distâncias com base nos pontos selecionados
def exibir_matriz_distancias(pontos_selecionados):
    if not pontos_selecionados:
        print("Nenhum ponto selecionado. Escolha os pontos primeiro.")
        return

    # Dicionário de abreviações personalizadas
    abreviacoes_personalizadas = {
        "Memorial Paulo Freire": "MPFreire",
        "Praça das Flores - UFERSA Angicos, RN": "P.Flores",
        "Restaurante Universitário": "Rest.Uni",
        "Biblioteca Campus Angicos": "Bibl.CA",
        "Bloco 1 de Aulas": "B1.Aulas",
        "Bloco 2 de Aulas": "B2.Aulas",
        "Bloco administrativo - UFERSA Angicos, RN": "Bloco.Adm",
        "Laboratórios 1 - UFERSA Angicos, RN": "Lab.1",
        "Laboratórios 2 - UFERSA Angicos, RN": "Lab.2",
        "Bloco 1 de Professores": "B1.Prof",
        "Bloco 2 de Professores": "B2.Prof",
        "Estacionamento": "Estac.",
        "Garagem da UFERSA Angicos": "Garagem",
        "Ginásio Poliesportivo da UFERSA Angicos": "Gin.Poli",
        "Almoxarifado da UFERSA Angicos": "Almox."
    }

    print("\nMatriz de Distâncias (em metros):")
    
    # Usar o dicionário de abreviações personalizadas ou gerar abreviação automática
    abreviacoes = {}
    for ponto in pontos_selecionados:
        abreviacoes[ponto] = abreviacoes_personalizadas.get(ponto, ''.join([palavra[0] for palavra in ponto.split()]))

    num_pontos = len(pontos_selecionados)
    tamanho_celula = 10  # Ajusta o tamanho das células da tabela
    
    # Desenha a linha superior da tabela
    print("┌" + "─" * tamanho_celula + ("┬" + "─" * tamanho_celula) * (num_pontos - 1) + "┐")

    # Imprime o cabeçalho da tabela
    header = "│"
    for ponto in pontos_selecionados:
        header += f"{abreviacoes[ponto]:^{tamanho_celula}}│"
    print(header)
        
    # Desenha a linha divisória após o cabeçalho
    print("├" + "─" * tamanho_celula + ("┼" + "─" * tamanho_celula) * (num_pontos - 1) + "┤")
        
    # Imprime as linhas da tabela
    for i, ponto1 in enumerate(pontos_selecionados):
        row = "│"
        for j, ponto2 in enumerate(pontos_selecionados):
            row += f"{distancias[list(pontos.values()).index(ponto1)][list(pontos.values()).index(ponto2)]:^{tamanho_celula}}│"
        print(row)
        if i < num_pontos - 1:
            # Desenha as linhas horizontais entre as linhas da tabela
            print("├" + "─" * tamanho_celula + ("┼" + "─" * tamanho_celula) * (num_pontos - 1) + "┤")
    
    # Desenha a linha inferior da tabela
    print("└" + "─" * tamanho_celula + ("┴" + "─" * tamanho_celula) * (num_pontos - 1) + "┘")

# Função para retirar pontos
def retirar_pontos(pontos_selecionados):
    print("\nPontos selecionados atualmente:")
    for i, ponto in enumerate(pontos_selecionados, 1):
        print(f"{i}: {ponto}")
    pontos_retirar = input("Digite os números dos pontos que deseja retirar (separados por vírgula): ").split(',')
    pontos_retirar = [int(i) - 1 for i in pontos_retirar if i.isdigit() and 0 < int(i) <= len(pontos_selecionados)]
    for i in sorted(pontos_retirar, reverse=True):
        pontos_selecionados.pop(i)
    return pontos_selecionados

# Função principal
def main():
    pontos_lista = list(pontos.values())
    pontos_selecionados = []

    while True:
        print("\n--- Menu Principal ---")
        print("1: Escolher pontos de entrega")
        print("2: Gerar rota")
        print("3: Exibir matriz de distâncias")
        print("4: Retirar pontos")
        print("5: Sair")
        opcao = input("Escolha uma opção: ")

        if opcao == "1":
            print("\nPontos disponíveis:")
            for i, ponto in enumerate(pontos_lista, 1):
                print(f"{i}: {ponto}")
            pontos_selecionados_str = input("Escolha os pontos que deseja percorrer (separados por vírgula): ").split(',')
            pontos_selecionados = [pontos_lista[int(i) - 1] for i in pontos_selecionados_str if i.isdigit() and 0 < int(i) <= len(pontos_lista)]
            if len(pontos_selecionados) == 0:
                print("Nenhum ponto válido selecionado.")
                continue
        elif opcao == "2":
            if len(pontos_selecionados) == 0:
                print("Nenhum ponto selecionado. Escolha os pontos primeiro.")
                continue
            melhor_rota_indices, pontos_selecionados = algoritmo_genetico(distancias, pontos_selecionados)
            # aqui eu preciso usar o pontos_selecionados_map para converter os indices de volta para os pontos
            melhor_rota = [pontos_selecionados[idx] for idx in melhor_rota_indices]
            print("\nMelhor Rota Encontrada:")
            for ponto in melhor_rota:
                print(ponto)
            print(f"\nDistância total: {calcular_distancia_total(melhor_rota_indices, distancias, pontos_selecionados)} metros")

        elif opcao == "3":
            exibir_matriz_distancias(pontos_selecionados)

        elif opcao == "4":
            if len(pontos_selecionados) == 0:
                print("Nenhum ponto selecionado para retirar.")
                continue
            pontos_selecionados = retirar_pontos(pontos_selecionados)

        elif opcao == "5":
            print("Saindo do programa...")
            break

        else:
            print("Opção inválida. Tente novamente.")

# Executar o programa
if __name__ == "__main__":
    main()
