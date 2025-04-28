+++
authors = ["Arthur Araujo"]
title = "Consertando o update de emails"
date = "2025-04-25"
description = "Um pequeno post sobre como resolvi o issue"
tags = [
    "wordview",
    "api",
    "spring",
    "java",
]
+++



Um pequeno post sobre como resolvi o issue [#14](https://github.com/word-view/APIWordView/issues/14) na API do WordView.


#### O Problema
O salt da senha é gerado através da combinação de `email + senha`, então atualizar somente
o email quebra o sistema de login, quando o usuário que alterou o seu email tentar novamente
entrar na conta o hash de login não vai ser igual ao armazenado no servidor.  

```java
@PutMapping("/me")
public ResponseEntity<?> update(@RequestBody UserUpdateRequest request, HttpServletRequest req) {
	return response(() -> {
		User user = service.getMe(req);
		User userAlter = request.toEntity();

		// o método merge escreve os valores de 'userAlter' sobre os de 'user'
		User merged = ClassMerger.merge(user, userAlter);

		// insert salva os valores alterados
		service.insert(merged);

		return ok();
	});
}
```

---

#### Solucionando
Eu comecei por criar um novo tipo de request e o teste para ele. Enquanto eu criava esse teste
percebi que todos os request tests são muito parecidos e que seria benéfico refatorar e simplicá-los, então criei um novo issue para trabalhar nessa questão depois.
```java
@Getter
@Setter
public class UserEmailUpdateRequest {
    private String oldEmail;
    private String newEmail;
    private String password;
// ...
}
```

---

Para manter o sistema simples decidi separar a atualização do email da atualização das
demais informações, juntar tudo em um só endpoint adicionaria uma complexidade a mais.

A idéia é simples: Eu crio dois hashes (com o novo e com o velho email), comparo se o velho 
está de acordo com o que está armazenado e se sim eu altero o email e a senha com os novos valores.

```java
@PutMapping("/me/email")
public ResponseEntity<?> updateEmail(@RequestBody UserEmailUpdateRequest request, HttpServletRequest req) {					
	return response(() -> {
		request.validate();

		User user = service.getMe(req);

		String oldHash = new HashedPassword(request.getOldEmail(), request.getPassword()).getValue();
		String newHash = new HashedPassword(request.getNewEmail(), request.getPassword()).getValue();

		if (user.getPassword().equals(oldHash)) {
			user.setEmail(request.getNewEmail());
			user.setPassword(newHash);
		} else {
			throw new IncorrectCredentialsException("Email or password did not match");
		}

		service.insert(user);

		return ok();
	});
}
```

___

O teste também é simples: Eu faço login na conta de teste, para garantir que o login esta funcionando corretamente,
altero o email da conta, e tento fazer o login novamente usando o novo email.

```java
class UserControllerEmailUpdateTest extends ControllerTest {
    @Test
    @Order(1)
    void login() throws Exception {
        MockUser user = new MockUser("mock.user@gmail.com", "S_enha64");

        req.post("/user/login", user.toJson()).andExpect(status().isOk());
    }

    @Test
    @Order(2)
    void updateEmail() throws Exception {
        String jwt = MockValues.getUserJwt(mockMvc);
        MockEmailUpdateRequest request = new MockEmailUpdateRequest("mock.user@gmail.com", "new.email@gmail.com", "S_enha64");

        req.put("/user/me/email", request.toJson(), jwt).andExpect(status().isOk());
    }

    @Test
    @Order(3)
    void loginNewEmail() throws Exception {
        MockUser user = new MockUser("new.email@gmail.com", "S_enha64");

        req.post("/user/login", user.toJson()).andExpect(status().isOk());
    }
}
```

---

Enquanto criava o teste percebi que faltava algo: Ver se o email novo já estava sendo usado por outra conta.

Por conta dessa lógica adicional, acabou ficando muito 'caro' manter a lógica toda no Controller, então
decidi mover tudo para o `UserService`

```java
@PutMapping("/me/email")
public ResponseEntity<?> updateEmail(@RequestBody UserEmailUpdateRequest request, HttpServletRequest req) {
	return response(() -> {
		request.validate();

		service.insertWithNewEmail(req, request.getNewEmail(), request.getOldEmail(), request.getPassword());

		return ok();
	});
}
```

```java
@Override
public User insertWithNewEmail(HttpServletRequest request, String newEmail, String oldEmail, String password) throws ValueTakenException, InvalidKeySpecException, NoSuchEntryException, IncorrectCredentialsException {
    Optional<User> existingUserWithNewEmail = repository.findByEmail(newEmail);

    if (existingUserWithNewEmail.isPresent()) {
        throw new ValueTakenException("newEmail is already taken by another account.");
    }

    User user = getMe(request);

    String oldHash = new HashedPassword(oldEmail, password).getValue();
    String newHash = new HashedPassword(newEmail, password).getValue();

    if (user.getPassword().equals(oldHash)) {
        user.setEmail(newEmail);
        user.setPassword(newHash);
    } else {
        throw new IncorrectCredentialsException("Email or password did not match");
    }

    return insert(user);
}
```

---

Com a nova variável, a ordem do teste fica assim: Login, tentar alterar o email com um email já em uso, alterar para um email novo, fazer login com o email novo.

```java
@Test
@Order(2)
void updateExistingEmail() throws Exception {
    String jwt = MockValues.getUserJwt(mockMvc);
    MockEmailUpdateRequest request = new MockEmailUpdateRequest("mock.user@gmail.com", "mock.admin@gmail.com", "S_enha64");

    req.put("/user/me/email", request.toJson(), jwt).andExpect(status().isForbidden());
}
```

---

Com isso, eu rodo todos os testes e pronto!