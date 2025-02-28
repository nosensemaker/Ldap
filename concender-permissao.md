
# Concedendo Permissão para Usuário Comum Trocar Senhas de Outros Usuários no LDAP

## 1. Acessar o Arquivo de Configuração do LDAP:

1.1. **Abrir o arquivo `slapd.conf`** em /etc/ldap/slapd.conf
   ```bash
   sudo vim /etc/ldap/slapd.conf
   ```

## 2. Definir as Regras de Acesso para Troca de Senha

2.1 Adicionar as regras de acesso para permitir que um **usuário comum** altere a senha de outros usuários. Isso envolve configurar permissões específicas para o atributo `userPassword` e para o **DN** dos usuários:

2.2 Adicionar a seguinte configuração de acesso para o atributo `userPassword`:

```ldif
access to attrs=userPassword
    by self write
    by dn.exact="cn=Marcelo Batista,ou=People,dc=in,dc=iti,dc=br" manage
    by anonymous auth
    by * none
```

- **`by self write`**: Permite que o próprio usuário modifique seu atributo `userPassword`.
- **`by dn.exact="cn=Marcelo Batista,ou=People,dc=in,dc=iti,dc=br" manage`**: Permite que o usuário `Marcelo Batista` gerencie (escreva) as senhas de outros usuários.
- **`by anonymous auth`**: Permite a autenticação de usuários anônimos para leitura de atributos públicos, como o login.
- **`by * none`**: Bloqueia o acesso de qualquer outro usuário que não tenha permissões específicas.

### 3. **Permitir Que Usuários Comuns Leiam Dados de Outros Usuários**

3.1 Configuração que permite que qualquer usuário comum possa ler os dados de outros usuários, mas não os altere. Isso é necessário para permitir que o usuário saiba quem são os outros usuários para quem ele pode alterar a senha:

```ldif
access to *
    by self write
    by users read
    by * none
```

- **`by self write`**: O usuário pode modificar seus próprios dados.
- **`by users read`**: Usuários podem ler dados de outros usuários, mas não podem modificá-los.
- **`by * none`**: Bloqueia o acesso a outros usuários e grupos.

### 4. **Configuração para um Usuário Específico Conseguir Visualizar ou Trocar a PRÓPRIA Senha**

4.1 Configuração de acesso para que um usuário específico tenha a capacidade de **alterar** ou **visualizar sua própria senha**, mas **não** a de outros usuários. A configuração se aplica à subárvore `ou=People,dc=in,dc=iti,dc=br`:

```ldif
access to dn.subtree="ou=People,dc=in,dc=iti,dc=br"
    by self write
    by dn="cn=Lethicia Miranda,ou=People,dc=in,dc=iti,dc=br" read
    by * none
```

- **`by self write`**: O usuário pode modificar seus próprios dados.
- **`by dn="cn=Lethicia Miranda,ou=People,dc=in,dc=iti,dc=br" read`**: O usuario especificado pode ler dados de outros usuários, mas não podem modificá-los.
- **`by * none`**: Bloqueia o acesso a outros usuários e grupos.


## 5. Reinicie o OpenLDAP para Aplicar as Alterações

5.2 Reiniciar o serviço do OpenLDAP:

```bash
sudo systemctl restart slapd
```

## 6. Testar manualmente.

6.1 Trocar pelo terminal a troca de senha: do usuario com privilegio e do que deseja trocar:

```ldif
ldappasswd -x -D "cn=Marcelo Batista,ou=People,dc=in,dc=iti,dc=br" -W -S "cn=Lethicia Miranda,ou=People,dc=in,dc=iti,dc=br
