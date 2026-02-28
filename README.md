# Binary-Instrumentation-2 Pico-CTF
I've been learning more Windows API functions to do my bidding. Hmm... I swear this program was supposed to create a file and write the flag directly to the file. Can you try and intercept the file writing function to see what went wrong?

## Exploration

O `bininst2.exe` se apresenta como um programa que deveria criar um arquivo e escrever a flag dentro dele, mas na prática nunca faz isso. A razão só fica clara quando você para de olhar para o que o binário *faz* e começa a olhar para o que ele *carrega*.

O executável principal funciona como um loader — seu verdadeiro propósito é desempacotar e executar um segundo binário que está escondido dentro dele, comprimido com LZMA na seção `.ATOM`. Essa seção não faz parte do padrão PE, o que já é um sinal imediato de que algo foi deliberadamente embutido ali. Os dados começam com o byte `0x5D`, que é a assinatura do formato LZMA, seguido das informações de tamanho e dicionário típicas desse algoritmo. Descomprimindo esse conteúdo, obtém-se um segundo executável Windows completo de 11 KB.

É nesse executável interno que a lógica real do desafio está. Ele importa `CreateFileA` e `WriteFile` do `KERNEL32.dll` — as funções que deveriam criar o arquivo e escrever a flag. O problema está no argumento passado para `CreateFileA`: o caminho do arquivo é literalmente a string `<Insert path here>`, um placeholder que o desenvolvedor esqueceu de substituir. Como esse caminho é inválido, a função falha silenciosamente, retorna um handle inválido, e `WriteFile` nunca consegue escrever nada. A flag simplesmente nunca chega ao disco.

O que torna o desafio interessante é que a flag não precisava ser interceptada em execução — ela já estava lá, embutida em Base64 diretamente no código do executável interno, esperando para ser lida. Decodificando a string, a flag aparece imediatamente.

A exploração inteira gira em torno de perceber que o binário entregue não é o binário que importa, e que o bug não está na lógica de escrita em si, mas num detalhe tão simples quanto um caminho de arquivo que nunca foi preenchido.

# Ghidra Analysis

<img width="700" alt="image" src="https://github.com/user-attachments/assets/a7ae5f30-503f-4e26-98be-d1ad6a1a4200" />

<img width="550" alt="image" src="https://github.com/user-attachments/assets/6e0cd689-552a-483e-8acc-d2faa4a7f930" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/7f96e4dd-f556-423b-8d45-5bb94b1acf9e" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/cbd3b21f-a9f2-460f-b4ad-11ace110e8c5" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/c630c0f9-44f9-4697-9973-48b92a48ae54" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/9086ca54-e7a4-4d10-b0de-c7d13a44d52d" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/a580b25a-e44e-4126-b742-da9c7187c7c7" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/d1b4a13b-ebf9-4dda-ab08-52d64824eb0a" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/9d372837-dba4-469c-a4c3-abe1a5f156b2" />

<img width="550" alt="image" src="https://github.com/user-attachments/assets/c905f89b-57bc-4937-95ee-bbc9235f3d3b" />

<img width="550" alt="image" src="https://github.com/user-attachments/assets/c0e1b434-0dcd-464c-bd7b-dce4225ebc98" />

<img width="550" alt="image" src="https://github.com/user-attachments/assets/da6d6154-07fc-406f-bd50-3570a7dc03a7" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/2ea1cb3e-1fe5-467f-9305-58a67ec63e75" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/881c5992-ee52-4d46-ba01-0ec93f89768c" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/1eaa8390-a21b-48cc-acf9-ae72e05406fb" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/fce9e108-620d-43ea-a072-32de74377424" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/94a5e7d6-480b-42a4-863f-621655a0e000" />

<img width="700" alt="image" src="https://github.com/user-attachments/assets/a6a0df63-58c0-42e7-942d-5c10c2081b67" />

