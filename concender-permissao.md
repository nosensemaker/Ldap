
# Concedendo Permissão para Usuário Comum Trocar Senhas de Outros Usuários no LDAP

Esta documentação orienta como configurar as permissões necessárias no OpenLDAP para permitir que um **usuário comum** tenha a capacidade de alterar a senha de outros usuários no diretório LDAP, sem precisar de acesso administrativo.

## Pré-requisitos

Antes de iniciar o processo, garanta que você tenha:

- Acesso de administrador ao servidor OpenLDAP.
- Familiaridade com o arquivo de configuração `slapd.conf`.
- O usuário desejado configurado no LDAP e com a capacidade de se autenticar corretamente.

## Passo 1: Acesse o Arquivo de Configuração do LDAP

O OpenLDAP armazena suas configurações no arquivo `slapd.conf`. Este arquivo define as regras de acesso ao diretório e as permissões atribuídas a usuários e grupos.

1. **Acesse o servidor LDAP via SSH**:
   ```bash
   ssh usuario@servidor_ldap
   ```

2. **Abra o arquivo `slapd.conf`** para edição:
   ```bash
   sudo nano /etc/ldap/slapd.conf
   ```

## Passo 2: Defina as Regras de Acesso para Troca de Senha

Agora, você precisará adicionar ou ajustar as regras de acesso para permitir que um **usuário comum** altere a senha de outros usuários. Isso envolve configurar permissões específicas para o atributo `userPassword` e para o **DN (Distinguished Name)** dos usuários.

### 1. **Permitir a Modificação da Senha pelo Próprio Usuário**

Para que um usuário possa alterar sua própria senha, adicione a seguinte configuração de acesso para o atributo `userPassword`:

```ldif
access to attrs=userPassword
    by self write
    by dn.exact="cn=Marcelo Batista,ou=People,dc=in,dc=iti,dc=br" manage
    by anonymous auth
    by * none
```

- **`by self write`**: Permite que o próprio usuário modifique seu atributo `userPassword`.
- **`by dn.exact="cn=Marcelo Batista,ou=People,dc=in,dc=iti,dc=br" manage`**: Permite que o usuário `Marcelo Batista` gerencie (escreva) as senhas de outros usuários. Este é um exemplo, e você pode personalizar o DN de acordo com o usuário ou grupo responsável pela gestão das senhas.
- **`by anonymous auth`**: Permite a autenticação de usuários anônimos para leitura de atributos públicos, como o login.
- **`by * none`**: Bloqueia o acesso de qualquer outro usuário que não tenha permissões específicas.

### 2. **Permitir Que Usuários Comuns Leiam Dados de Outros Usuários**

Esta configuração permite que qualquer usuário comum possa ler os dados de outros usuários, mas não os altere. Isso é necessário para permitir que o usuário saiba quem são os outros usuários para quem ele pode alterar a senha.

```ldif
access to *
    by self write
    by users read
    by * none
```

- **`by self write`**: O usuário pode modificar seus próprios dados.
- **`by users read`**: Usuários podem ler dados de outros usuários, mas não podem modificá-los.
- **`by * none`**: Bloqueia o acesso a outros usuários e grupos.

### 3. **Permitir a Leitura e Modificação de Senhas em Subárvores Específicas (Exemplo com `ou=People`)**

No exemplo abaixo, configuramos o acesso para que um usuário específico possa ler e modificar senhas de outros usuários dentro da subárvore `ou=People,dc=in,dc=iti,dc=br`:

```ldif
access to dn.subtree="ou=People,dc=in,dc=iti,dc=br"
    by self write
    by dn="cn=Lethicia Miranda,ou=People,dc=in,dc=iti,dc=br" read
    by * none
```

- **`by self write`**: Permite que o próprio usuário altere dados dentro da subárvore.
- **`by dn="cn=Lethicia Miranda,ou=People,dc=in,dc=iti,dc=br" read`**: Permite que o usuário `Lethicia Miranda` leia dados dentro da subárvore `ou=People`, mas não faça alterações.
- **`by * none`**: Bloqueia o acesso de outros usuários que não têm permissão específica.

## Passo 3: Reinicie o OpenLDAP para Aplicar as Alterações

Após realizar as alterações no `slapd.conf`, é necessário reiniciar o serviço do OpenLDAP para que as novas permissões entrem em vigor.

```bash
sudo systemctl restart slapd
```

## Passo 4: Teste as Novas Permissões

Agora que as permissões estão configuradas, é importante testar se os usuários têm a capacidade de alterar a senha de outros usuários.

1. **Autentique-se como um usuário comum** (por exemplo, `Lethicia Miranda`):
   ```bash
   ldapsearch -x -D "cn=Lethicia Miranda,ou=People,dc=in,dc=iti,dc=br" -W -b "ou=People,dc=in,dc=iti,dc=br" "(cn=*)" 
   ```

2. **Tente alterar a senha de outro usuário** com a ferramenta `ldappasswd` ou utilizando o **phpLDAPadmin**.
   - **Comando de Alteração de Senha**:
     ```bash
     ldappasswd -D "cn=Lethicia Miranda,ou=People,dc=in,dc=iti,dc=br" -W -S "uid=usuario_alvo,ou=People,dc=in,dc=iti,dc=br"
     ```

3. **Verifique o comportamento no phpLDAPadmin**:
   - Acesse o **phpLDAPadmin** com o usuário que recebeu a permissão de modificar as senhas.
   - Tente modificar a senha de outros usuários e verifique se a ação é bem-sucedida.

## Passo 5: Monitore e Ajuste as Permissões conforme Necessário

Lembre-se de monitorar o uso dessas permissões. Caso você queira limitar ainda mais o acesso, poderá ajustar as regras de acesso de acordo com a necessidade, como permitir a modificação apenas de uma lista específica de usuários.

---

# Considerações Finais

Conceder permissão a um **usuário comum** para trocar a senha de outros usuários no OpenLDAP é um processo que envolve a configuração cuidadosa das regras de acesso. Com as permissões adequadas, você poderá garantir que o processo seja seguro e eficiente, sem comprometer a segurança do sistema.

Sempre revise as permissões periodicamente e, se necessário, ajuste conforme a necessidade de acesso e segurança de sua infraestrutura LDAP.
