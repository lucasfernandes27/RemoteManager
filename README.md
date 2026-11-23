# Remote Manager

Aplicativo desktop local para organizar e acessar conexões SSH, SFTP e RDP em um só lugar. Os dados dos hosts e o histórico são armazenados em um banco SQLite no computador.

> **Estado do projeto:** em desenvolvimento. A integração RDP está em validação; depende do controle de Área de Trabalho Remota instalado no Windows e ainda precisa ser conferida em diferentes versões do Windows e configurações de servidor.

## Recursos

- Cadastro, edição, busca e organização de hosts por protocolo, grupo, ambiente, favoritos e tags.
- Terminal SSH interativo.
- Navegação SFTP com transferência de arquivos.
- Integração de área de trabalho RDP dentro da janela, usando o controle RDP do Windows (experimental).
- Histórico local de conexões, com opção para apagá-lo.
- Exportação e importação de backup protegido por senha.
- Banco local SQLite com índices para as listagens mais usadas.

## Requisitos

- Windows 10 ou 11, 64 bits.
- Para usar RDP, o Windows precisa ter o componente de Área de Trabalho Remota (`mstscax.dll`) disponível e o host remoto precisa aceitar conexões RDP.
- Para desenvolver e compilar: Node.js 24 ou superior e npm.
- Para compilar o helper RDP: compilador C# do .NET Framework 4.x em `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe`.

## Instalação pelo GitHub

Quando houver uma versão publicada, baixe o instalador MSI na seção **Releases** do repositório e execute-o no Windows. Também é possível gerar o instalador localmente com os passos abaixo.

## Desenvolvimento

Clone o repositório, entre na pasta do projeto e instale as dependências:

```powershell
git clone https://github.com/lucasfernandes27/RemoteManager.git
cd RemoteManager
npm ci
```

Inicie o aplicativo em modo de desenvolvimento:

```powershell
npm run dev
```

Comandos adicionais:

```powershell
npm run typecheck # Verifica os tipos TypeScript
npm test          # Executa os testes automatizados
npm run build     # Compila o aplicativo e o helper RDP
npm run dist      # Gera o instalador MSI em release/
```

O helper RDP é compilado por `build-rdp-helper.ps1`. Se o compilador C# não estiver no caminho esperado, ajuste esse script para apontar para o `csc.exe` instalado no ambiente.

## Benchmark local

O benchmark usa um banco temporário e dados sintéticos. Ele não acessa hosts cadastrados nem serviços externos. Por padrão, cria até 50 mil hosts e executa a etapa de duração por 60 segundos; a duração máxima configurada é 10 minutos.

```powershell
npm run benchmark
```

Para alterar o volume e a duração:

```powershell
$env:BENCH_HOSTS = '10000'
$env:BENCH_SOAK_SECONDS = '30'
npm run benchmark | Out-File -Encoding utf8 benchmark.json
```

## Dados e segurança

- As senhas, chaves privadas SSH e frases secretas são protegidas com `safeStorage` do Electron. Elas não são devolvidas ao renderer depois de salvas.
- O banco SQLite guarda localmente os metadados dos hosts e o histórico. Esses metadados não recebem criptografia adicional do aplicativo.
- Na primeira conexão SSH, confira a impressão digital da chave do servidor por um canal confiável antes de aceitá-la. O aplicativo salva a chave aceita e rejeita alterações posteriores.
- Os backups usam AES-256-GCM com chave derivada por scrypt. A senha do backup deve ter pelo menos oito caracteres. Guarde-a separadamente do arquivo.
- As credenciais protegidas pelo Windows podem depender da conta que as salvou. Não presuma que elas serão utilizáveis ao importar o banco em outro computador ou usuário.
- No RDP, confira os avisos de certificado e prossiga apenas se reconhecer e confiar no servidor. Para uso permanente, prefira certificados configurados pela equipe responsável pela infraestrutura.

Não publique bancos locais, arquivos de backup, logs, credenciais ou chaves privadas no repositório. O `.gitignore` exclui dependências, saídas `out/` e `release/`, logs e arquivos de banco `.db`.

## Tecnologias

- Electron, React, TypeScript e Vite.
- SQLite com `better-sqlite3`.
- SSH e SFTP com `ssh2`.
- RDP pelo controle ActiveX do Windows (`MsRdpClient` / `mstscax.dll`), hospedado em um helper C#.

## Licença

O `package.json` declara a licença MIT. Inclua um arquivo `LICENSE` no repositório ao publicar para disponibilizar o texto da licença junto ao código.
