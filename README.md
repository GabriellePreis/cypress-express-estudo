# cypress-express-estudo
Cypress

/// <reference types="cypress" />

describe ('tarefas', ()=>{
 
    //Cenário 1
    it('Deve cadastrar uma nova tarefa', ()=>{

           //Utilizamos a requisição http DELETE passando pela rota helper em ambiente de teste para excluir a tarefa antes de rodar o teste
           //Devido a regra de negócio, não podemos ter tarefas com nomes iguais
           //Poderíamos utilizar dados dinâmicos, mas iria poluir o ambiente

        cy.request({
            url: 'http://localhost:3333/helper/tasks',
            method: 'DELETE',
            body: {name:'Fazer um desenho para pintar com giz'}
        }).then(response => {
            expect(response.status).to.eq(204)
        })


        cy.visit('http://localhost:3000')

        cy.get('input[placeholder= "Add a new Task"]')
            .type('Fazer um desenho para pintar com giz')

        cy.contains('button', 'Create').click()

        cy.contains('main div p', 'Fazer um desenho para pintar com giz')
           .should('be.visible')
    })
       
    //Cenário 2
    it('Não deve permitir tarefa duplicada', ()=> {

        cy.request({
            url: 'http://localhost:3333/helper/tasks',
            method: 'DELETE',
            body: {name:'Ler um capítulo do livro'}
        }).then(response => {
            expect(response.status).to.eq(204)
        })

        //Dado que eu tenho uma tarefa duplicada

         cy.request({
            url: 'http://localhost:3333/tasks',
            method: 'POST', 
            body: {name:'Ler um capítulo do livro', is_done:false}
         }).then(response => {
            expect(response.status).to.eq(201)
         })

         
        //Quando faço o cadastro da tarefa com o mesmo nome

        cy.visit('http://localhost:3000')

        cy.get('input[placeholder= "Add a new Task"]')
            .type('Ler um capítulo do livro')

        cy.contains('button', 'Create').click()
        
        //Então vejo a mensagem de duplicidade

        cy.get('.swal2-html-container')
            .should('be.visible')
            .should('have.text', 'Task already exists!')



    })

})
