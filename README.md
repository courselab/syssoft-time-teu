# Membros:

Adrio Oliveira Alves, 11796830

Eduardo Vinicius Barbosa Rossi, 10716887

Pedro Augusto Ribeiro Gomes, 11819125

Thiago Henrique Cardoso, 11796594


# Sobre:

Nesse README explicaremos o desenvolvimento da questão 1, da qual os arquivos estão no repositorio ``/hack01``.

O desenvolvimento das questões 2 e extra estão no pdf neste repositório.


## Explicação da descoberta da vulnerabilidade:

Inicialmente, enfrentamos o problema de autenticação e procuramos contorná-lo. Descobrimos que poderíamos explorar o fluxo lógico do código binário usando o programa gdb. Utilizamos comandos como "disassemble" para visualizar a função principal (main) e percebemos a chamada da função "authorize". Também usamos outros comandos, como "ni" (next instruction), "si" (step instruction), "set disassembly-favor intel" (configura a visualização em formato Intel), "record" (registra a execução para reprodução posterior), "fin" (finish), "print" (imprimir).

Em seguida, examinamos o código de máquina reconstruído (binários docrypt e libauth.so) usando o programa objdump para identificar quais variáveis, registradores e parâmetros da pilha estavam sendo usados para autenticar o usuário. Observamos que a chamada da função "authorize" realizava alguma forma de verificação e, no final, retornava um valor inteiro como confirmação da validade do acesso do usuário.

Descobrimos que poderíamos contornar esse procedimento emulando um valor de retorno verdadeiro, independentemente da autenticação real do usuário. Em outras palavras, se a lógica que segue a função "authorize" retornar apenas o inteiro de confirmação da autenticação (independentemente de ter sido realmente autenticado), o restante do sistema poderá realizar a descriptografia sem problemas.

Observamos que o registrador EAX armazena o status da função "authorize".

## Explicação do *bypass*:

Devido ao fato de a biblioteca "libauth.so" ser chamada dinamicamente e a linkagem dinâmica depender apenas do nome da função, sem verificar sua assinatura ou os parâmetros que ela recebe, podemos redirecionar a chamada da função "authorize" usada pelo "docrypt" para uma nova versão que sempre retorna um valor verdadeiro de autorização. Isso nos permite obter sempre a autenticação, independentemente do status real.

A diretiva "run" do arquivo Makefile redireciona automaticamente para essa nova versão da função "authorize", sem que o usuário precise estar ciente desse mecanismo ao utilizar a ferramenta.

## Explicação da recuperação da chave de descriptografia:

Devido ao fato de a chave de criptografia ser uma string estática armazenada no código do "docrypt", exploramos essa vulnerabilidade de design para examinar a seção da ABI (Application Binary Interface) reservada para armazenamento de dados estáticos. Usando a saída do comando "objdump -x .rodata docrypt", identificamos várias strings de prompt e, entre elas, a chave ("easy").

A execução do comando "run" do programa Makefile substitui automaticamente a chave de descriptografia pela chave encontrada. No entanto, é possível especificar outra chave, se necessário.

## Recomendações técnicas para melhorias nos aspectos de robustez e segurança do aplicativo:

Primeiramente, é essencial revisar o sistema de autenticação, garantindo que seja implementado de forma segura, com validação adequada das credenciais do usuário e proteção contra ataques de força bruta e injeção de código. Além disso, é recomendável utilizar uma abordagem de autenticação baseada em tokens ou criptografia assimétrica para evitar a exposição de senhas em texto claro.

Em relação ao bypass, é importante fortalecer a segurança da aplicação por meio da implementação de verificações adicionais de integridade e autenticidade dos componentes utilizados, como a biblioteca libauth.so. A assinatura digital dos arquivos pode ser uma solução viável para garantir que apenas versões confiáveis sejam carregadas e executadas. Além disso, é fundamental realizar testes de segurança e revisões de código regulares para identificar e corrigir possíveis pontos fracos, como a não verificação adequada dos parâmetros de função durante a linkagem dinâmica.

Para melhorar a segurança da chave de decriptação, recomenda-se evitar o armazenamento direto de chaves estáticas no código. Em vez disso, é aconselhável utilizar mecanismos seguros de gerenciamento de chaves, como o armazenamento em um local protegido ou o uso de serviços externos de gerenciamento de chaves. Isso reduz o risco de exposição da chave e dificulta sua recuperação por meio de análise de código. Além disso, é importante implementar técnicas de ofuscação de código para tornar a extração de informações sensíveis mais difícil para potenciais atacantes.


