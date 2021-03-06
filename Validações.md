# BestMeal - Validações

## Por que Validar no lado cliente?

> A validação do lado do cliente fornece ao usuário um feedback imediato, sem ter que esperar o carregamento da página. As aplicações cliente modernas fazem a validação imediata das entradas de dados, ou seja, à medida que os dados são digitados por você. Antigamente era preciso teclar um botão `gravar` ou `enviar` para que o lado cliente informasse quais os campos estavam inconsistentes. Infelizmente muitas aplicações ainda são até hoje!

## A validação do lado cliente é suficente?

> Infelizmente não. Se o browser tiver os scripts desabilitado (por exemplo, JavaScript desabilitado), a validação não será disparada, e é por isso que você precisa do servidor para verificar os valores também.

## Quais os tipos de validação são utilizadas no Angular?

> O Angular permite duas formas de validação em formulários: `Template-driven` e `Reactive Forms`.

### Template-driven

> `Template-driven` usa atributos de validação exatamente como você como faria com a validação de formulário HTML nativo. Angular usa diretivas para combinar esses atributos com funções validadoras no framework. Toda vez que o valor de um controle de formulário for alterado, o Angular executa a validação e gera uma lista de erros de validação, que resulta em um status INVALID ou nulo.

### Exemplo de `Template-drive`

```typescript
<input id="name" name="name" class="form-control"
      required minlength="4" appForbiddenName="bob"
      [(ngModel)]="hero.name" #name="ngModel" >

<div *ngIf="name.invalid && (name.dirty || name.touched)"
    class="alert alert-danger">

  <div *ngIf="name.errors.required">
    Name is required.
  </div>
  <div *ngIf="name.errors.minlength">
    Name must be at least 4 characters long.
  </div>
  <div *ngIf="name.errors.forbiddenName">
    Name cannot be Bob.
  </div>

</div>
```

<p align="center">
   <strong>Listagem 1- Exemplo de validação usando Template-drive</strong> 
</p>

### Reactive Forms

> `Reactive Forms` Nessa modalidade, em vez de adicionar validadores por meio de atributos no modelo, você adiciona funções do validador diretamente ao modelo de controle de formulário na classe do componente. Angular então chama essas funções sempre que o valor do controle é alterado.
> Funções do validador

Existem dois tipos de funções validadoras: validadores síncronos e validadores assíncronos.

    Validadores síncronos: retornam imediatamente um conjunto de erros de validação ou nulo.

    Validadores assíncronos: retornam um Promise ou Observable que posteriormente emite um conjunto de erros de validação ou null.

> Nota 1: por motivos de desempenho, o Angular só executa validadores assíncronos se todos os validadores de síncronos passarem. Cada um deve concluir antes que os erros sejam definidos.

> Nota 2: o Jhipster a partir da versão 6.X usa `Reactive Forms` por padrão.

#### Implementando a primeira validação do tipo `Reactive Forms` na aplicação BestMeal

> A validação que iremos implementar, impede que uma determina string seja utilizada no nome.

::: :walking: Passo a passo :::

1. Cria a estrutura de pastas conforme Figura 1

<p align="center">
  <img src="images/EstruturaValidators_50.png" alt="Estrutura de pastas para validadores">
</p>
<p align="center">
   <strong>Figura 1- Imagem da estrutura de pastas para validadores</strong> 
</p>

2. Na nova pasta, crie um novo arquivo denominado `custom-name.service.ts`, conforme Listagem 2

```typescript
import { AbstractControl, ValidatorFn } from '@angular/forms';
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class CustomNameValidatorService {
  forbiddenNameValidator(nameRe: RegExp): ValidatorFn {
    return (control: AbstractControl): { [key: string]: any } => {
      const forbidden = nameRe.test(control.value);
      return forbidden ? { jhiForbiddenName: { value: control.value } } : null;
    };
  }
}
```

<p align="center">
   <strong>Liustagem 2 -custom-name.service.ts </strong> 
</p>

3. Altere o arquivo denominado `cartao-credito-update.component.ts`, conforme Listagem 3

```typescript
...
import { CustomNameValidatorService } from '../../shared/validators/custom-name.service'; // >>> Alterado aqui
...

@Component({
  selector: 'jhi-cartao-credito-update',
  templateUrl: './cartao-credito-update.component.html'
})
export class CartaoCreditoUpdateComponent implements OnInit {
  cartaoCredito: ICartaoCredito;
  isSaving: boolean;

  editForm = this.fb.group({
    id: [],
    nomeCartao: [
      null,
      [Validators.required, Validators.minLength(10), Validators.maxLength(40)
      ,this.nameValidatorService.forbiddenNameValidator(/bosco/i)] // >>> Alterado aqui
    ],
    bandeira: [],
...
 constructor(
    protected cartaoCreditoService: CartaoCreditoService,
    protected activatedRoute: ActivatedRoute,
    private fb: FormBuilder,
    protected nameValidatorService: CustomNameValidatorService // >>> Alterado aqui
  ) {}

  ngOnInit() {
  ...
```

<p align="center">
   <strong>Listagem 3 -cartao-credito-update.components.ts </strong> 
</p>

4. Altere o arquivo denominado `cartao-credito-update.component.html`, conforme Listagem 4

```html
                <div class="form-group">
                    <label class="form-control-label" jhiTranslate="bestMealApp.cartaoCredito.nomeCartao" for="field_nomeCartao">Nome Cartao</label>
                    <input type="text" class="form-control" name="nomeCartao" id="field_nomeCartao"
                           formControlName="nomeCartao"/>
                        ...
                        <!-- bloco de código adicionado -->
                        <small class="form-text text-danger"
                               *ngIf="editForm.get('nomeCartao').errors.jhiForbiddenName" jhiTranslate="entity.validation.forbiddenName"
                               [translateValues]="{ forbiddenName: 'bosco' }">
                        This field cannot be longer than 40 characters.
                        </small>
                    </div>
                </div>

```

<p align="center">
   <strong>Listagem 4 -cartao-credito-update.components.html </strong> 
</p>

5. Altere o arquivo denominado `global.json`, conforme Listagem 5

```json
    "validation": {
      "required": "O campo é obrigatório.",


      // Adicionadas as 2 linhas abaixo
      "forbiddenName": "Este campo não pode conter a palavra {{forbiddenName}}",
      "validadeCart": "Mês e ano tem que ter um valor maior ou igual a {{mesanoMin}}"
    }
```

<p align="center">
   <strong>Listagem 5 -arquivo global.json </strong> 
</p>

6. Pronto. Execute agora a aplicação com

```java

mvn

```

7. Testando a aplicação

<p align="center">
  <img src="images/ValidacaoNome.png" alt="Validação de nome no cartão">
</p>
<p align="center">
   <strong>Figura 2- Imagem da Validação de nome no cartão</strong> 
</p>

#### Implementando a segunda validação do tipo `Reactive Forms` na aplicação BestMeal

> A segunda validação que iremos implementar, verifica a validade do cartão de crédito.

::: :walking: Passo a passo :::

1. Na nova pasta `shared/validators`, crie um novo arquivo denominado `custom-date.service.ts`, conforme Listagem 6

```typescript
import { AbstractControl, ValidatorFn } from '@angular/forms';
import { Injectable } from '@angular/core';

/**
 * Classe para fazer validações com datas
 *
 */

@Injectable({ providedIn: 'root' })
export class CustomDateValidatorService {
  /**
   *
   * @param mesano //mes e ano mínimo
   * Este método valida se mm/yyyy informado é maior ou igual a um mínimo
   */
  expireDateValidator(mesano: string): ValidatorFn {
    return (control: AbstractControl): { [key: string]: any } => {
      const mesMin = mesano.substring(0, 2);
      const anoMin = mesano.substring(3, 7);
      if (control.value) {
        // se nao for nulo
        const mesInf = control.value.substring(0, 2);
        const anoInf = control.value.substring(3, 7);
        const isInvalid = anoInf < anoMin || (anoInf === anoMin && mesInf < mesMin);
        return isInvalid ? { validadeCart: { value: control.value } } : null;
      } else {
        return null;
      }
    };
  }
}
```

<p align="center">
   <strong>Listagem 6 -arquivo custom-date.service.ts </strong> 
</p>

2. Altere o arquivo denominado `cartao-credito-update.component.ts`, conforme Listagem 7

```typescript

import { CustomDateValidatorService } from '../../shared/validators/custom-date-service';// >>> Alterado aqui

...

export class CartaoCreditoUpdateComponent implements OnInit {

  mesano: string = moment().format('MM/YYYY'); // >> alterado aqui

  editForm = this.fb.group({
    id: [],
    ...
    validade: [
      null,
      this.dateValidateService.expireDateValidator(this.mesano)] // >> Alterado aqui
    ]
  });
...
```

<p align="center">
   <strong>Listagem 7 -cartao-credito-update.components.ts </strong> 
</p>

3. Altere o arquivo denominado `cartao-credito-update.component.html`, conforme Listagem 8

```html
                <div class="form-group">
                    <label class="form-control-label" jhiTranslate="bestMealApp.cartaoCredito.validade" for="field_validade">Validade</label>
                    <input type="text" class="form-control" name="validade" id="field_validade"
                           formControlName="validade"/>

                        <!-- bloco de código adicionado -->
                    <small class="form-text text-danger"
                               *ngIf="editForm.get('validade').errors.validadeCart" jhiTranslate="entity.validation.validadeCart" [translateValues]="{ mesanoMin: mesano}">
                            This field should follow pattern for "Validade".
                    </small>
                    </div>
                </div>

```

<p align="center">
   <strong>Listagem 8 -cartao-credito-update.components.html </strong> 
</p>

4. Pronto. Execute agora a aplicação com

```java

mvn

```

5. Testando a aplicação

<p align="center">
  <img src="images/ValidacaoExpira.png" alt="Validação da data que o cartão expira">
</p>
<p align="center">
   <strong>Figura 2- Imagem da validação da data que o cartão expira</strong> 
</p>
