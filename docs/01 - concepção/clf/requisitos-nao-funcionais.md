
# Requisitos não funcionais do Alume

### RNF001 — Usabilidade

- **Descrição:** O sistema deve apresentar uma interface simples, clara e intuitiva, permitindo que o estudante identifique rapidamente a situação de frequência de cada disciplina.

### RNF002 — Multiplataforma

- **Descrição:** O sistema deve funcionar em navegadores web modernos e oferecer compatibilidade com dispositivos móveis, podendo ser instalado e utilizado como Progressive Web App (PWA).

### RNF003 — Segurança na autenticação

- **Descrição:** O sistema deve utilizar autenticação integrada ao SUAP, sem armazenar a senha do usuário em nenhum banco de dados próprio. O token de acesso obtido do SUAP deve ficar sob responsabilidade do backend, armazenado temporariamente no Redis com tempo de expiração, e nunca ser enviado ao navegador. A sessão do usuário deve ser mantida por um JWT próprio do Alume, guardado em cookie HttpOnly.

### RNF004 — Dependência da API institucional e do armazenamento de sessões

- **Descrição:** O funcionamento do sistema deve depender da disponibilidade da API do SUAP, uma vez que as informações acadêmicas são consultadas diretamente na plataforma institucional, e da disponibilidade do Redis, utilizado como banco de dados próprio temporário para as sessões.

### RNF005 — Performance

- **Descrição:** O sistema deve apresentar as informações de disciplinas e frequência em tempo adequado, evitando atrasos perceptíveis durante a utilização.

### RNF006 — Manutenibilidade

- **Descrição:** O sistema deve possuir uma arquitetura organizada, com separação de responsabilidades entre autenticação, integração com a API institucional, processamento das informações e apresentação dos dados.
