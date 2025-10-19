# Guia de Contribuição - Curso Completo de Pacemaker Linux

Obrigado por considerar contribuir para o **Curso Completo de Pacemaker Linux**! 🎉

Este documento fornece diretrizes e instruções para contribuir com o projeto. Leia com atenção antes de começar.

---

## 📋 Índice

1. Código de Conduta
2. Como Contribuir]
3. Processo de Contribuição
4. Diretrizes de Estilo
5. Processo de Review
6. Dúvidas Frequentes

---

## 🤝 Código de Conduta

### Nossa Promessa

No interesse de promover um ambiente aberto e acolhedor, nós, como colaboradores e mantenedores, nos comprometemos a tornar a participação em nosso projeto e nossa comunidade uma experiência livre de assédio para todos, independentemente de idade, tamanho corporal, deficiência, etnia, identidade e expressão de gênero, nível de experiência, nacionalidade, aparência pessoal, raça, religião ou identidade e orientação sexual.

### Nossos Padrões

Exemplos de comportamento que contribuem para criar um ambiente positivo incluem:

- Usar linguagem acolhedora e inclusiva
- Ser respeitoso com pontos de vista e experiências divergentes
- Aceitar críticas construtivas com graça
- Focar no que é melhor para a comunidade
- Mostrar empatia com outros membros da comunidade

Exemplos de comportamento inaceitável incluem:

- Uso de linguagem ou imagens sexualizadas e atenção sexual indesejada
- Trolling, comentários insultuosos/depreciativos e ataques pessoais ou políticos
- Assédio público ou privado
- Publicar informações privadas de outras pessoas, como endereço físico ou eletrônico, sem permissão explícita
- Outra conduta que poderia razoavelmente ser considerada inadequada em um ambiente profissional

### Aplicação

Instâncias de comportamento abusivo, de assédio ou inaceitável podem ser reportadas entrando em contato com a equipe do projeto. Todas as reclamações serão revisadas e investigadas.

---

## 💡 Como Contribuir

Existem muitas formas de contribuir para este projeto:

### 1. Reportar Bugs

Encontrou um erro no conteúdo ou nos exercícios? Abra uma [Issue](https://github.com/fabiobilac/Curso-Completo-de-Pacemaker-Linux/issues) com:

- **Título claro:** Descreva o problema em uma frase
- **Descrição detalhada:** Explique o que esperava vs. o que aconteceu
- **Passos para reproduzir:** Como alguém pode reproduzir o problema
- **Contexto:** Qual módulo, qual exercício, qual versão do SO
- **Screenshots/Logs:** Se aplicável, inclua evidências

**Exemplo:**
```
Título: Erro no Módulo 03 - Comando de criação de cluster falha

Descrição:
Ao seguir o Módulo 03, o comando `pcs cluster setup` retorna erro de autenticação.

Passos para reproduzir:
1. Seguir os passos 1-3 do Módulo 03
2. Executar: sudo pcs cluster setup mycluster node1 node2
3. Erro: "Authentication failed"

Contexto:
- Ubuntu 22.04 LTS
- Pacemaker 2.1.5
- Corosync 3.1.5

Logs:
[cole aqui o erro completo]
```

### 2. Sugerir Melhorias

Tem uma ideia para melhorar o curso? Abra uma [Discussion](https://github.com/fabiobilac/Curso-Completo-de-Pacemaker-Linux/discussions) com:

- **Descrição clara:** O que você gostaria de ver
- **Justificativa:** Por que isso seria útil
- **Exemplos:** Como isso poderia funcionar
- **Impacto:** Quem se beneficiaria

**Exemplo:**
```
Título: Adicionar exercício com banco de dados PostgreSQL

Descrição:
Seria ótimo ter um exercício que mostrasse como usar Pacemaker 
com um banco de dados PostgreSQL em alta disponibilidade.

Justificativa:
Muitas empresas usam PostgreSQL, e um exemplo prático ajudaria 
muito os alunos a entender como aplicar Pacemaker em produção.

Exemplo:
Um exercício que configure:
- Cluster com 2 nós
- PostgreSQL com replicação streaming
- Pacemaker gerenciando failover automático
- Testes de failover e recuperação
```

### 3. Melhorar Documentação

Encontrou um typo, frase confusa ou exemplo que poderia ser melhor? Contribua com melhorias:

- Correção de typos e erros gramaticais
- Clarificação de conceitos confusos
- Melhoria de exemplos
- Adição de diagramas ou imagens
- Tradução para outros idiomas

### 4. Adicionar Novos Exercícios

Quer criar um novo exercício prático? Excelente! Siga o padrão dos exercícios existentes:

- **Cenário realista:** Descreva um problema real
- **Pré-requisitos:** O que é necessário
- **Passos detalhados:** Instruções passo a passo
- **Resultados esperados:** O que deve acontecer
- **Desafios de aprofundamento:** Tarefas opcionais mais complexas
- **Checklist:** Forma de verificar conclusão

### 5. Criar Scripts de Automação

Quer automatizar tarefas do laboratório? Contribua com scripts que:

- Automatizem a criação de VMs
- Configurem o ambiente automaticamente
- Testem o cluster
- Façam backup e restore

**Requisitos:**
- Bem documentados
- Testados
- Suportam múltiplas distribuições (Ubuntu, CentOS, RHEL)

### 6. Traduzir para Outros Idiomas

Quer traduzir o curso para outro idioma? Maravilhoso!

- Comece com um módulo
- Mantenha a estrutura original
- Adapte exemplos para o contexto local
- Inclua um arquivo `README.[IDIOMA].md`

---

## 🔄 Processo de Contribuição

### Passo 1: Fork o Repositório

```bash
# Vá para https://github.com/fabiobilac/Curso-Completo-de-Pacemaker-Linux
# Clique em "Fork" no canto superior direito
```

### Passo 2: Clone seu Fork

```bash
git clone https://github.com/SEU_USUARIO/Curso-Completo-de-Pacemaker-Linux.git
cd Curso-Completo-de-Pacemaker-Linux
```

### Passo 3: Crie uma Branch

```bash
# Para correções
git checkout -b fix/descricao-do-problema

# Para novas features
git checkout -b feature/descricao-da-feature

# Para melhorias de documentação
git checkout -b docs/descricao-da-melhoria

# Para traduções
git checkout -b translate/idioma-nome-modulo
```

**Exemplos de nomes de branch:**
- `fix/typo-modulo-01`
- `feature/novo-exercicio-postgresql`
- `docs/melhorar-secao-stonith`
- `translate/spanish-modulo-02`

### Passo 4: Faça suas Mudanças

Edite os arquivos conforme necessário. Siga as Diretrizes de Estilo.

### Passo 5: Teste suas Mudanças

Antes de enviar:

```bash
# Se for um exercício, teste os passos
# Se for documentação, verifique a formatação Markdown
# Se for um script, teste em múltiplas distribuições
```

### Passo 6: Commit suas Mudanças

```bash
# Adicione os arquivos
git add .

# Faça um commit com mensagem clara
git commit -m "Tipo: Descrição breve

Descrição mais detalhada se necessário.
Explique o quê e por quê, não o como.

Fixes #123 (se aplicável)
"
```

**Tipos de commit:**
- `fix:` Correção de bug
- `feature:` Nova funcionalidade
- `docs:` Mudanças na documentação
- `style:` Formatação, sem mudanças de lógica
- `refactor:` Reorganização de código
- `test:` Adição de testes
- `chore:` Tarefas de manutenção

**Exemplos:**
```
fix: Corrige comando de criação de cluster no Módulo 03

docs: Adiciona seção de troubleshooting no README

feature: Adiciona exercício com PostgreSQL HA

translate: Traduz Módulo 02 para Espanhol
```

### Passo 7: Push para seu Fork

```bash
git push origin nome-da-sua-branch
```

### Passo 8: Abra um Pull Request

1. Vá para o repositório original
2. Clique em "New Pull Request"
3. Selecione sua branch
4. Preencha o template do PR com:
   - **Título:** Descrição clara
   - **Descrição:** Por que essa mudança é necessária
   - **Tipo:** Bug fix, Feature, Docs, etc.
   - **Checklist:** Marque os itens completados

**Template de PR:**
```markdown
## Descrição
Descreva brevemente as mudanças.

## Tipo de Mudança
- [ ] Bug fix (correção de problema)
- [ ] Feature (nova funcionalidade)
- [ ] Docs (mudança na documentação)
- [ ] Outro (especifique)

## Checklist
- [ ] Testei as mudanças
- [ ] Segui as diretrizes de estilo
- [ ] Atualizei a documentação
- [ ] Não há conflitos com a branch main

## Relacionado
Fixes #123 (se aplicável)
```

### Passo 9: Responda aos Reviews

Mantenedores podem solicitar mudanças. Seja respeitoso e colaborativo:

```bash
# Faça as mudanças solicitadas
# Commit novamente
git commit -m "Responde aos comentários do review"

# Push (não precisa fazer novo PR)
git push origin nome-da-sua-branch
```

### Passo 10: Merge

Após aprovação, um mantenedor fará o merge. Parabéns! 🎉

---

## 📝 Diretrizes de Estilo

### Markdown

- Use **Markdown** para toda documentação
- Máximo 80 caracteres por linha (quando possível)
- Use headings hierarquicamente: `#` → `##` → `###`
- Use listas com `-` ou `*` (consistente)
- Use backticks para código inline: `` `comando` ``
- Use blocos de código com linguagem: ` ```bash `

**Exemplo:**
```markdown
## Seção Principal

Parágrafo explicativo com `código inline`.

### Subseção

Lista de itens:
- Item 1
- Item 2
- Item 3

Bloco de código:
```bash
comando aqui
```
```

### Exercícios Práticos

Siga o padrão dos exercícios existentes:

```markdown
## Exercício XX: Título Descritivo

**Módulo:** Módulo XX  
**Nível:** Intermediário/Avançado  
**Tempo Estimado:** XX-XX minutos  
**Objetivo:** Descrição clara do objetivo

### Cenário
Descrição do cenário realista...

### Pré-requisitos
- Pré-requisito 1
- Pré-requisito 2

### Passos Detalhados

#### Passo 1: Descrição (XX minutos)

Instruções...

```bash
comando 1
comando 2
```

**Resultado Esperado:**
```
saída esperada
```

### Desafios de Aprofundamento

1. Desafio 1
2. Desafio 2

### Checklist de Conclusão

- [ ] Item 1
- [ ] Item 2
```

### Comandos e Código

- Sempre teste os comandos antes de submeter
- Use comentários para explicar comandos complexos
- Mostre a saída esperada
- Indique quando um comando é específico para Ubuntu/CentOS/RHEL

**Exemplo:**
```bash
# 1. Instalar pacotes (Ubuntu)
sudo apt install -y pacemaker corosync pcs

# 2. Ou para CentOS/RHEL:
# sudo dnf install -y pacemaker corosync pcs

# 3. Iniciar serviço
sudo systemctl start pcsd
```

### Correções de Typos

- Corrija typos em português (não em nomes de comandos)
- Mantenha a formatação original
- Se for uma mudança grande, considere um PR separado

---

## 🔍 Processo de Review

### O que Esperamos

- **Qualidade:** Conteúdo bem pesquisado e testado
- **Clareza:** Fácil de entender para iniciantes
- **Completude:** Instruções passo a passo
- **Consistência:** Segue o padrão do projeto
- **Utilidade:** Agrega valor ao curso

### Critérios de Aceitação

Para um PR ser aceito:

- ✅ Segue as diretrizes de estilo
- ✅ Testado e funcionando
- ✅ Documentação clara
- ✅ Sem conflitos com main
- ✅ Aprovado por pelo menos um mantenedor

### Feedback Construtivo

Críticas são sempre construtivas e respeitosas:

- ❌ "Isso está errado"
- ✅ "Poderia ser melhorado assim... porque..."

- ❌ "Seu código é ruim"
- ✅ "Sugiro refatorar assim para melhor legibilidade"

---

## ❓ Dúvidas Frequentes

### P: Preciso de permissão para contribuir?

**R:** Não! Qualquer pessoa pode fazer um fork e enviar um PR. Você não precisa ser "convidado".

### P: Meu PR foi rejeitado. E agora?

**R:** Não é pessoal! Peça feedback específico e tente novamente. A maioria dos PRs é aceita após ajustes.

### P: Quanto tempo leva para revisar um PR?

**R:** Geralmente 3-7 dias. Se demorar mais, comente no PR para chamar atenção.

### P: Posso trabalhar em múltiplas coisas ao mesmo tempo?

**R:** Sim! Use branches diferentes para cada mudança. Isso facilita o gerenciamento.

### P: E se eu não souber como fazer algo?

**R:** Abra uma Discussion! A comunidade está aqui para ajudar.

### P: Posso traduzir o curso?

**R:** Sim! Comece com um módulo e abra um PR. Coordenaremos com você.

### P: Como faço para virar mantenedor?

**R:** Contribua regularmente com qualidade. Mantenedores são escolhidos entre colaboradores ativos.

### P: Posso usar o conteúdo em outro lugar?

**R:** Sim, desde que respeite a licença MIT. Dê crédito ao projeto original.

---

## 🎯 Áreas Procuradas

Estamos especialmente interessados em:

- [ ] **Novos Exercícios Práticos**
  - Exercícios com bancos de dados (MySQL, PostgreSQL)
  - Exercícios com containers (Docker, Kubernetes)
  - Exercícios com aplicações web reais

- [ ] **Melhorias de Documentação**
  - Diagramas e imagens
  - Vídeos tutoriais
  - Guias de troubleshooting

- [ ] **Scripts de Automação**
  - Provisionar ambiente de laboratório
  - Testar cluster
  - Backup/restore

- [ ] **Suporte para Múltiplas Distribuições**
  - Debian
  - Fedora
  - Alpine Linux

---

## 📞 Contato

Tem dúvidas ou quer discutir algo?

- 💬 Abra uma [Discussion](https://github.com/fabiobilac/Curso-Completo-de-Pacemaker-Linux/discussions)
- 📧 Abra uma [Issue](https://github.com/fabiobilac/Curso-Completo-de-Pacemaker-Linux/issues)
- 🌐 Consulte a [Documentação](https://clusterlabs.org/pacemaker/doc/)

---

## 📚 Recursos Úteis

- [GitHub Docs - Contributing](https://docs.github.com/en/get-started/quickstart/contributing-to-projects)
- [Git Cheat Sheet](https://github.github.com/training-kit/downloads/github-git-cheat-sheet.pdf)
- [Markdown Guide](https://www.markdownguide.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)

---

## 🙏 Agradecimentos

Obrigado por considerar contribuir! Sua ajuda torna este curso melhor para todos. 

**Contribuidores são o coração de projetos de código aberto.** ❤️

---

## 📄 Licença

Ao contribuir para este projeto, você concorda que suas contribuições serão licenciadas sob a mesma licença MIT do projeto.

---

## Resumo Rápido

1. **Fork** o repositório
2. **Clone** seu fork
3. **Crie uma branch** com nome descritivo
4. **Faça suas mudanças** seguindo as diretrizes
5. **Commit** com mensagem clara
6. **Push** para seu fork
7. **Abra um Pull Request** no repositório original
8. **Responda aos reviews** se necessário
9. **Merge** após aprovação

**Obrigado por contribuir!** 🎉
