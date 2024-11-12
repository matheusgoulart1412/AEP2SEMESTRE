#include <stdio.h>
#include <locale.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <ctype.h>

void Criptografia(char senha[20]) {
    int i = 0;
    while (senha[i]) {
        int incremento = 7;
        senha[i] += incremento;
        i++;
    }
}

int validarRA(const char *ra) {
    if (strlen(ra) != 10) return 0;
    for (int i = 0; i < 8; i++) {
        if (!isdigit(ra[i])) return 0;  
    }
    return ra[8] == '-' && isdigit(ra[9]);  
}

int verificarSenhaForte(const char *senha) {
    int temLetra = 0, temNumero = 0, temEspecial = 0;
    for (int i = 0; senha[i] != '\0'; i++) {
        if (isalpha(senha[i])) temLetra = 1;
        else if (isdigit(senha[i])) temNumero = 1;
        else temEspecial = 1;
    }
    return temLetra && temNumero && temEspecial && strlen(senha) >= 7 && strlen(senha) <= 16;
}

void limparTela() {
    system("cls");  
    printf("-=-=-= BEM VINDO =-=-=-\n");
    printf("     Sistema da AEP\n");
    printf("Opções:\n 1. Incluir Cadastro.\n 2. Excluir Cadastro.\n 3. Alterar Cadastro.\n 4. Listar Todos os Cadastros.\n 5. Créditos.\n 6. Sair.\n\n");
}

void aguardarUsuario() {
    printf("\nPressione Enter para voltar ao menu principal...");
    while (getchar() != '\n');  
}

void salvarCadastro(const char *ra, const char *senhaCripto) {
    FILE *arquivo = fopen("Cadastro.txt", "a");

    if (arquivo) {
        fprintf(arquivo, "RA: %s\nSenha: %s\n", ra, senhaCripto);  
        fclose(arquivo);
    } else {
        printf("Erro ao abrir o arquivo!\n");
    }
}

void listarCadastros() {
    FILE *arquivo = fopen("Cadastro.txt", "r");

    if (arquivo) {
        char linha[50];
        printf("-=-=-= LISTA =-=-=-\nLista Atualizada\n");
        while (fgets(linha, sizeof(linha), arquivo) != NULL) {
            printf("%s", linha);
        }
        fclose(arquivo);
    } else {
        printf("Erro ao abrir o arquivo!\n");
    }
    aguardarUsuario();
}

void excluirCadastro(const char *ra) {
    FILE *arquivo = fopen("Cadastro.txt", "r");
    FILE *temp = fopen("TempCadastro.txt", "w");

    if (arquivo && temp) {
        char linha[50];
        int encontrado = 0;

        while (fgets(linha, sizeof(linha), arquivo) != NULL) {
            if (strncmp(linha, "RA: ", 4) == 0 && strncmp(linha + 4, ra, strlen(ra)) == 0) {
                fgets(linha, sizeof(linha), arquivo);  
                fgets(linha, sizeof(linha), arquivo);  
                encontrado = 1;
                continue;  
            }
            fputs(linha, temp); 
        }

        fclose(arquivo);
        fclose(temp);

        remove("Cadastro.txt");
        rename("TempCadastro.txt", "Cadastro.txt");

        printf(encontrado ? "\nCadastro excluído com sucesso!\n" : "\nCadastro não encontrado!\n");
    } else {
        printf("Erro ao abrir os arquivos!\n");
    }
    aguardarUsuario();
}

void alterarCadastro(const char *raAntigo) {
    FILE *arquivo = fopen("Cadastro.txt", "r");
    FILE *temp = fopen("TempCadastro.txt", "w");

    if (arquivo && temp) {
        char linha[50];
        char raNovo[11], senhaNova[17];
        int encontrado = 0;

        while (fgets(linha, sizeof(linha), arquivo) != NULL) {
            if (strncmp(linha, "RA: ", 4) == 0 && strncmp(linha + 4, raAntigo, strlen(raAntigo)) == 0) {
                fgets(linha, sizeof(linha), arquivo);  
                fgets(linha, sizeof(linha), arquivo);  
                encontrado = 1;

                do {
                    printf("Digite o novo RA (formato XXXXXXXX-X): ");
                    scanf("%10s", raNovo);
                    getchar();
                    if (!validarRA(raNovo)) printf("Formato de RA inválido! Tente novamente.\n");
                } while (!validarRA(raNovo));

                do {
                    printf("Digite a nova senha (entre 7 e 16 caracteres): ");
                    scanf("%16s", senhaNova);
                    getchar();
                    if (!verificarSenhaForte(senhaNova))
                        printf("Senha inválida! A senha deve ter entre 7 e 16 caracteres, letras, números e caracteres especiais.\n");
                    else
                        break;
                } while (1);

                Criptografia(senhaNova);

                fprintf(temp, "RA: %s\nSenha: %s\n", raNovo, senhaNova);
            } else {
                fputs(linha, temp);  
            }
        }

        fclose(arquivo);
        fclose(temp);

        remove("Cadastro.txt");
        rename("TempCadastro.txt", "Cadastro.txt");

        printf(encontrado ? "\nCadastro alterado com sucesso!\n" : "\nCadastro não encontrado!\n");
    } else {
        printf("Erro ao abrir os arquivos!\n");
    }
    aguardarUsuario();
}

int main() {
    setlocale(LC_ALL, "Portuguese");
    int op;
    char ra[11], senha[17];

    while (1) {
        limparTela();
        printf("Opção Escolhida: ");
        scanf("%d", &op);
        getchar();  

        switch(op) {
            case 1:
                limparTela();
                printf("-=-=-= INCLUIR CADASTRO =-=-=-\n");
                do {
                    printf("RA (formato XXXXXXXX-X): ");
                    scanf("%10s", ra);
                    getchar();  
                    if (!validarRA(ra)) printf("Formato de RA inválido! Tente novamente.\n");
                } while (!validarRA(ra));

                do {
                    printf("Senha (entre 7 e 16 caracteres): ");
                    scanf("%16s", senha);
                    getchar();  
                    if (!verificarSenhaForte(senha))
                        printf("Senha inválida! A senha deve ter entre 7 e 16 caracteres, letras, números e caracteres especiais.\n");
                    else
                        break;
                } while (1);

                printf("\nLogin incluído com sucesso!\n");
                Criptografia(senha); 
                salvarCadastro(ra, senha);  
                aguardarUsuario();
                break;

            case 2:
                limparTela();
                printf("-=-=-= EXCLUIR CADASTRO =-=-=-\n");
                do {
                    printf("RA (formato XXXXXXXX-X): ");
                    scanf("%10s", ra);
                    getchar();  
                    if (!validarRA(ra)) printf("Formato de RA inválido! Tente novamente.\n");
                } while (!validarRA(ra));

                excluirCadastro(ra);
                break;

            case 3:
                limparTela();
                printf("-=-=-= ALTERAR CADASTRO =-=-=-\n");
                do {
                    printf("Digite o RA atual para alteração: ");
                    scanf("%10s", ra);
                    getchar();
                    if (!validarRA(ra)) printf("Formato de RA inválido! Tente novamente.\n");
                } while (!validarRA(ra));

                alterarCadastro(ra);
                break;

            case 4:
                limparTela();
                listarCadastros();
                break;

            case 5:
                limparTela();
                printf("-=-=-=-=-=-=-= CRÉDITOS =-=-=-=-=-=-=-\nTrabalho feito por:\n");
                printf("Matheus Goulart de Alencar\nThiago Teodoro da Conceição\nEmanuelly Prestes Lopes\n");
                printf("Trabalho da AEP\n-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
                aguardarUsuario();
                break;

            case 6:
                limparTela();
                printf("-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
                printf("Muito obrigado por compilar nosso programa!!\nAté uma próxima!\n");
                printf("-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
                return 0;

            default:
                printf("\nOpção inválida! Tente novamente.\n");
                aguardarUsuario();
                break;
        }
    }

    return 0;
}
