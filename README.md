# YouTube-Web-Scraping
Script de web scraping para o YouTube

Um script de web scraping para o YouTube, focado em monitoramento contínuo, extração e transcrição de áudio dos vídeos.

## Descrição

Este script é uma ferramenta de coleta de dados robusta, projetada para monitorar o YouTube. Ele utiliza o  `yt-dlp` para buscar e baixar vídeos com base em uma lista de palavras-chave pré-definida.

O principal diferencial do projeto é seu pipeline de análise de mídia integrado, que emprega o modelo **Whisper da OpenAI** para transcrever o áudio dos vídeos coletados para texto. O script é desenvolvido para operação autônoma e contínua, com um sistema de persistência que evita a recoleta de conteúdo já processado.

## Principais Funcionalidades

-   **Monitoramento Contínuo:** Opera em um ciclo infinito, realizando buscas periódicas para capturar novos vídeos em tempo real.
-   **Transcrição de Áudio com IA:**
    -   Utiliza o modelo **OpenAI Whisper** para gerar transcrições de alta qualidade do áudio dos vídeos.
-   **Extração de Dados Rica:** Coleta o título do vídeo, metadados de publicação (data, URL), informações do canal (nome, ID) e métricas de engajamento (respostas, curtidas, visualizações).
-   **Lógica de Coleta Robusta:**
    -   **Persistência de Coleta:** Mantém um registro de todos os IDs de vídeos já processados para garantir que cada vídeo seja coletado apenas uma vez.
    -   **Tratamento de Erros:** Se o download ou a transcrição de um vídeo falhar, o script o ignora, permitindo que uma nova tentativa seja feita em um ciclo futuro. Isso garante que apenas dados 100% processados com sucesso sejam salvos.
-   **Privacidade (LGPD):** Pseudonimiza os IDs dos canais usando um hash SHA256 com "salt" para proteger a identidade dos criadores de conteúdo.
-   **Configuração Flexível:**
    -   Os termos de busca são gerenciados por um arquivo externo (`lista_de_buscas.txt`).
    -   O caminho para a dependência (FFmpeg) pode ser configurado diretamente no script, sem necessidade de alterar o PATH do sistema.
    -   Parâmetros como o número de resultados por busca e o intervalo entre os ciclos são facilmente ajustáveis.

## Como Funciona

O algoritmo opera em um ciclo contínuo com a seguinte lógica:

1.  **Inicialização:** O script verifica a presença do FFmpeg (primeiro no PATH do sistema, depois em um caminho manual), carrega o modelo Whisper, os termos de busca do arquivo de configuração e a lista de IDs de vídeos já coletados.

2.  **Início do Ciclo de Busca:** Para cada termo de busca definido em `lista_de_buscas.txt`:
    -   O `yt-dlp` é executado para buscar os vídeos mais recentes correspondentes ao termo, retornando seus metadados em formato JSON.

3.  **Processamento dos Vídeos:** Para cada vídeo novo encontrado:
    -   O script verifica se o ID do vídeo já existe no arquivo de persistência. Se sim, ele é ignorado.
    -   O vídeo é baixado para uma pasta temporária usando `yt-dlp`.
    -   **Se o download falhar (por exemplo, por timeout ou erro de rede), o vídeo é pulado** para ser tentado novamente no próximo ciclo.
    -   Se o download for bem-sucedido, o áudio é transcrito com o modelo Whisper.
    -   **Se a transcrição falhar, o vídeo também é pulado**, garantindo a integridade dos dados.
    -   O arquivo de mídia local é deletado (se configurado) para economizar espaço.

4.  **Salvamento:** Apenas os vídeos processados com 100% de sucesso (download + transcrição) são compilados em um objeto JSON e salvos em um novo arquivo na pasta `Coletas_Youtube`. Seus IDs são então adicionados ao arquivo de persistência.

5.  **Pausa:** O script aguarda o intervalo de tempo definido e inicia um novo ciclo de coleta.

## Estrutura do JSON de Saída

Os dados são salvos como uma lista de objetos JSON. Cada objeto representa um vídeo e segue a estrutura a seguir:

```json
[
    {
        "metadados_coleta": {
            "plataforma_postagem": "X",
            "data_coleta": "2025-09-29T19:35:10.123456Z",
            "coletado_via": "web_scraping",
            "termo_busca_utilizado": "Brasil OR \"Tecnologia da Informação\""
        },
        "dados_postagem": {
            "id_post": "1234567890123456789",
            "url": "[https://x.com/usuario_exemplo/status/1234567890123456789](https://x.com/usuario_exemplo/status/1234567890123456789)",
            "data_publicacao": "2025-09-29T18:45:00Z",
            "autor": {
                "id_usuario": null,
                "id_pseudonimizado": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2",
                "nome_usuario": "usuario_exemplo",
                "nome_exibicao": "Usuário de Exemplo",
                "perfil_verificado": true,
                "contagem_seguidores": null
            }
        },
        "engajamento": {
            "contagem_respostas": 15,
            "contagem_reposts": 32,
            "contagem_curtidas": 128,
            "contagem_visualizacoes": null
        },
        "conteudo": {
            "texto_principal": "Discutindo o futuro...!",
            "anexos": [
                {
                    "tipo_midia": "video",
                    "url_midia": "https://twiiit.com/usuario_exemplo/status/1234567890123456789",
                    "transcricao": "Olá a todos, no vídeo de hoje..."
                }
            ]
        }
    }
]
```
