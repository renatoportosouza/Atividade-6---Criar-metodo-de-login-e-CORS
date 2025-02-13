# Desenvolvimento de API com Autenticação e CORS

Este projeto tem como objetivo a implementação de um método de login e a configuração de CORS em uma API .NET, utilizando autenticação via JWTBearer para permitir operações restritas.

## Requisitos
- **Editor de Código**: [Visual Studio Code](https://code.visualstudio.com)
- **SDK do .NET**: Versão 6 ou 7
- **SQL Server Management Studio (SSMS)**: Para criar e gerenciar o banco de dados
- **Ferramenta para testes de API**: Insomnia ou Postman

## Passo a Passo

### 1. Configuração do Banco de Dados
1. Abra o **SQL Server Management Studio (SSMS)**.
2. No menu **Arquivo > Abrir Arquivo...**, localize e abra o arquivo `cria-db.sql`.
3. Clique em **Executar** para criar o banco de dados e os usuários.
   - Se já existir um banco de dados com o mesmo nome, exclua-o antes de executar o script.

### 2. Configuração do Projeto no Visual Studio Code
1. Baixe e extraia o arquivo `ATIVIDADE-06.zip`.
2. Abra o terminal no diretório `Exo.WebApi` e verifique a versão do .NET:
   ```sh
   dotnet --version
   ```
   - Certifique-se de que a versão instalada é **6 ou superior**.
3. Abra o projeto no VSCode com o comando:
   ```sh
   code .
   ```

### 3. Implementação do Método de Login
1. Acesse o arquivo `UsuarioRepository.cs` e insira o seguinte código caso ainda não esteja presente:
   ```csharp
   public Usuario Login(string email, string senha)
   {
       return _context.Usuarios.FirstOrDefault(u => u.Email == email && u.Senha == senha);
   }
   ```
2. No arquivo `Exo.WebApi.csproj`, adicione a seguinte referência ao pacote `JwtBearer` caso ainda não esteja presente:
   ```xml
   <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="6.0.0"/>
   ```
3. No terminal, execute o comando para restaurar os pacotes:
   ```sh
   dotnet restore
   ```

### 4. Configuração do Controle de Acesso e Token
1. No arquivo `UsuariosController.cs`, importe os pacotes necessários:
   ```csharp
   using Microsoft.IdentityModel.Tokens;
   using System.IdentityModel.Tokens.Jwt;
   using System.Security.Claims;
   ```
2. Adicione a implementação do método POST para autenticação:
   ```csharp
   public IActionResult Post(Usuario usuario)
   {
       Usuario usuarioBuscado = _usuarioRepository.Login(usuario.Email, usuario.Senha);
       if (usuarioBuscado == null)
       {
           return NotFound("E-mail ou senha inválidos!");
       }
       var claims = new[]
       {
           new Claim(JwtRegisteredClaimNames.Email, usuarioBuscado.Email),
           new Claim(JwtRegisteredClaimNames.Jti, usuarioBuscado.Id.ToString()),
       };
       var key = new SymmetricSecurityKey(System.Text.Encoding.UTF8.GetBytes("exoapi-chave-autenticacao"));
       var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
       var token = new JwtSecurityToken(
           issuer: "exoapi.webapi",
           audience: "exoapi.webapi",
           claims: claims,
           expires: DateTime.Now.AddMinutes(30),
           signingCredentials: creds
       );
       return Ok(new { token = new JwtSecurityTokenHandler().WriteToken(token) });
   }
   ```

### 5. Configuração do CORS e Autenticação no `Program.cs`
1. No arquivo `Program.cs`, adicione a configuração de autenticação JWT:
   ```csharp
   builder.Services.AddAuthentication(options =>
   {
       options.DefaultAuthenticateScheme = "JwtBearer";
       options.DefaultChallengeScheme = "JwtBearer";
   })
   .AddJwtBearer("JwtBearer", options =>
   {
       options.TokenValidationParameters = new TokenValidationParameters
       {
           ValidateIssuer = true,
           ValidateAudience = true,
           ValidateLifetime = true,
           IssuerSigningKey = new SymmetricSecurityKey(System.Text.Encoding.UTF8.GetBytes("exoapi-chave-autenticacao")),
           ClockSkew = TimeSpan.FromMinutes(30),
           ValidIssuer = "exoapi.webapi",
           ValidAudience = "exoapi.webapi"
       };
   });
   ```
2. Habilite a autenticação e a autorização:
   ```csharp
   app.UseAuthentication();
   app.UseAuthorization();
   ```

### 6. Testes com Insomnia/Postman
1. Execute o projeto:
   ```sh
   dotnet run
   ```
2. No Insomnia ou Postman, tente atualizar um usuário sem um token (PUT em `https://localhost:7154/api/usuarios/2`). O resultado deve ser `401 Unauthorized`.
3. Para gerar um token, envie uma requisição POST com credenciais válidas:
   ```json
   {
       "email": "email@sp.br",
       "senha": "1234"
   }
   ```
4. Copie o token retornado e use-o para autenticar solicitações protegidas (PUT/DELETE).
5. Para adicionar autenticação Bearer no Insomnia/Postman:
   - Acesse a guia **Auth**.
   - Escolha **Bearer Token**.
   - Cole o token copiado.
6. Agora, ao tentar atualizar ou deletar um usuário, o retorno deve ser `204 No Content` (sucesso).

## Conclusão
Este tutorial guiou a implementação do método de login, autenticação via JWTBearer e configuração do CORS na API. Após seguir os passos, sua API estará protegida e apta a realizar operações seguras.
