#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char placa[8];
    char motivo[100];
    char dataApreensao[11];
    char local[100];
} CarroApreendido;

typedef struct {
    CarroApreendido *data;  // Ponteiro para o array de carros
    int size;                // Número de carros na lista
    int capacity;            // Capacidade máxima do array
} CarrosApreendidos;

void adicionarCarro(CarrosApreendidos *carros, CarroApreendido novo) {
    if (carros->size == carros->capacity) {
        expandirCarros(carros);  // Expande a capacidade se necessário
    }
    carros->data[carros->size] = novo;
    carros->size++;
}
void editarCarro(CarrosApreendidos *carros) {
    char placa[8];
    printf("Digite a placa do carro a ser editado: ");
    fgets(placa, sizeof(placa), stdin);
    strtok(placa, "\n");

    for (int i = 0; i < carros->size; i++) {
        if (strcmp(carros->data[i].placa, placa) == 0) {
            printf("Digite o novo motivo: ");
            fgets(carros->data[i].motivo, sizeof(carros->data[i].motivo), stdin);
            strtok(carros->data[i].motivo, "\n");

            printf("Digite a nova data de apreensão (DD/MM/AAAA): ");
            fgets(carros->data[i].dataApreensao, sizeof(carros->data[i].dataApreensao), stdin);
            strtok(carros->data[i].dataApreensao, "\n");

            printf("Digite o novo local: ");
            fgets(carros->data[i].local, sizeof(carros->data[i].local), stdin);
            strtok(carros->data[i].local, "\n");
            printf("Dados do carro editados com sucesso!\n");
            return;
        }
    }
    printf("Carro com a placa %s não encontrado.\n", placa);
}
void exibirCarros(CarrosApreendidos *carros) {
    for (int i = 0; i < carros->size; i++) {
        printf("\nCarro %d:\n", i + 1);
        printf("Placa: %s\n", carros->data[i].placa);
        printf("Motivo: %s\n", carros->data[i].motivo);
        printf("Data Apreensão: %s\n", carros->data[i].dataApreensao);
        printf("Local: %s\n", carros->data[i].local);
    }
}
void salvarEmArquivo(CarrosApreendidos *carros, const char *nomeArquivo) {
    FILE *arquivo = fopen(nomeArquivo, "w");
    if (arquivo == NULL) {
        perror("Erro ao abrir arquivo para salvar");
        return;
    }
    // Escreve os dados no formato CSV
    for (int i = 0; i < carros->size; i++) {
        fprintf(arquivo, "%s,%s,%s,%s\n", carros->data[i].placa, carros->data[i].motivo,
                carros->data[i].dataApreensao, carros->data[i].local);
    }
    fclose(arquivo);
    printf("Dados salvos no arquivo %s.\n", nomeArquivo);
}
void carregarDeArquivo(CarrosApreendidos *carros, const char *nomeArquivo) {
    FILE *arquivo = fopen(nomeArquivo, "r");
    if (arquivo == NULL) {
        printf("Nenhum arquivo encontrado. Iniciando com lista vazia.\n");
        return;
    }
    CarroApreendido carro;
    while (fscanf(arquivo, "%7[^,],%99[^,],%10[^,],%99[^\n]\n", carro.placa, carro.motivo, 
            carro.dataApreensao, carro.local) == 4) {
        adicionarCarro(carros, carro);
    }
    fclose(arquivo);
    printf("Dados carregados do arquivo %s.\n", nomeArquivo);
}
int main() {
    CarrosApreendidos carros;
    initCarrosApreendidos(&carros);
    carregarDeArquivo(&carros, "carros.csv");

    char opcao;
    do {
        printf("\n1. Adicionar carro\n2. Editar carro\n3. Exibir todos\n4. Salvar e sair\nEscolha: ");
        opcao = getchar();
        while (getchar() != '\n');  // Limpa buffer

        switch (opcao) {
            case '1': 
                adicionarCarro(&carros, criarCarroApreendido()); 
                break;
            case '2': 
                editarCarro(&carros); 
                break;
            case '3': 
                exibirCarros(&carros); 
                break;
            case '4': 
                salvarEmArquivo(&carros, "carros.csv"); 
                break;
            default: 
                printf("Opção inválida.\n");
        }
    } while (opcao != '4');

    free(carros.data);
    return 0;
}
