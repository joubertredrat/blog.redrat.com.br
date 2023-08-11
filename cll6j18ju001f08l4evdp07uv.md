---
title: "Testes unitários para Custom Validation no Symfony"
datePublished: Fri Aug 11 2023 11:50:42 GMT+0000 (Coordinated Universal Time)
cuid: cll6j18ju001f08l4evdp07uv
slug: testes-unitarios-para-custom-validation-no-symfony
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/aPNzyCwdBvY/upload/822aaa048489e34a458511bb21bcca68.jpeg
tags: unit-testing, php, testing, symfony, php8

---

*Disclaimer: Eu não sou uma entidade divina. O que eu falo não é uma verdade absoluta. Não tenha medo de questionar até o mundo, pois ele pode estar errado, não você.*

Hoje, uma das coisas mais importantes para qualquer projeto é ter testes automatizados, de preferência, unitários. Eu gosto de seguir uma lógica própria que é:

> Todo código que você cria deve ter testes automatizados

Tendo isto em mente, vamos falar de testes unitários para Custom Validation que você cria em seu projeto do Symfony.

### Estou sem paciência, mostre me o código!

Okay okay! Caso você não esteja com paciência para ler este artigo, eu tenho um projeto de teste com a implementação deste artigo no link abaixo.

[https://github.com/joubertredrat/symfony-request-validation](https://github.com/joubertredrat/symfony-request-validation)

### Custom Validation

Um dos recursos mais antigos e funcionais do Symfony, é o componente de [Validation](https://symfony.com/doc/6.2/validation.html), que é validação dos dados de entrada na sua aplicação. Embora no próprio Symfony já tem suporte a [várias constraints](https://symfony.com/doc/6.2/validation.html#supported-constraints), em casos específcios é possível criar suas próprias [constraints customizadas](https://symfony.com/doc/6.2/validation/custom_constraint.html), como no exemplo abaixo.

```php
<?php declare(strict_types=1);

namespace App\Validator;

use Symfony\Component\Validator\Constraint;
use Symfony\Component\Validator\ConstraintValidator;
use Symfony\Component\Validator\Exception\UnexpectedTypeException;
use Symfony\Component\Validator\Exception\UnexpectedValueException;

class CreditCardValidator extends ConstraintValidator
{
    private const REGEX = '/^(4|5){1}[0-9]{15}$/';

    public function validate(mixed $value, Constraint $constraint): void
    {
        if (!$constraint instanceof CreditCard) {
            throw new UnexpectedTypeException($constraint, CreditCard::class);
        }

        if (null === $value || '' === $value) {
            return;
        }

        if (!is_string($value)) {
            throw new UnexpectedValueException($value, 'string');
        }

        if (!preg_match(self::REGEX, $value)) {
            $this
                ->context
                ->buildViolation($constraint->message)
                ->setParameter('{{ string }}', $value)
                ->addViolation()
            ;
        }
    }
}
```

No Validator acima, é feita uma validação de um número de cartão de crédito que deve conter 16 números começando com o número 4 ou 5.

### Teste unitário do Custom Validation

Para o nosso teste unitário, iremos utilizar o famoso [PHPUnit](https://phpunit.de/), como no exemplo abaixo.

```php
<?php declare(strict_types=1);

namespace App\Tests\Validator;

use App\Validator\CreditCard;
use App\Validator\CreditCardValidator;
use Symfony\Component\Validator\Constraints\Blank;
use Symfony\Component\Validator\ConstraintValidatorInterface;
use Symfony\Component\Validator\Exception\UnexpectedTypeException;
use Symfony\Component\Validator\Test\ConstraintValidatorTestCase;

class CreditCardValidatorTest extends ConstraintValidatorTestCase
{
    protected function createValidator(): ConstraintValidatorInterface
    {
        return new CreditCardValidator();
    }

    public function testValidateValueWithSuccess(): void
    {
        $this->validator->validate('5234567890123456', new CreditCard());
        $this->assertNoViolation();
    }

    public function testValidateWithInvalidConstraint(): void
    {
        $this->expectException(UnexpectedTypeException::class);
        $this->validator->validate('foo', new Blank());
    }

    public function testValidateWithInvalidValue(): void
    {
        $constraint = new CreditCard();
        $this->validator->validate('1234567890123456', $constraint);

        $this
            ->buildViolation($constraint->message)
            ->setParameter('{{ value }}', '1234567890123456')
            ->assertRaised()
        ;
    }
}
```

Analizando o exemplo acima, parece ser um típico teste unitário, porém, ele tem pequenas diferenças. Normalmente, as classes de teste é uma extensão da classe `PHPUnit\Framework\TestCase`, porém, no nosso caso, é uma extensão de `Symfony\Component\Validator\Test\ConstraintValidatorTestCase`, pois o componente de [Validation](https://symfony.com/doc/6.2/validation.html) já tem uma classe de TestCase com todas as dependências necessárias para fazer os testes unitários. Ao usar `ConstraintValidatorTestCase`, será necessário implementar a função `createValidator` retornando a classe de constraint relacionada a classe de validation e pronto. Depois disto, só criar todos os asserts necessários e executar os testes.

```bash
$ composer run tests
> vendor/phpunit/phpunit/phpunit --testdox
PHPUnit 10.3.1 by Sebastian Bergmann and contributors.

Runtime:       PHP 8.2.8
Configuration: phpunit.xml.dist

...                                                                 3 / 3 (100%)

Time: 00:00.017, Memory: 10.00 MB

Credit Card Validator (App\Tests\Validator\CreditCardValidator)
 ✔ Validate value with success
 ✔ Validate with invalid constraint
 ✔ Validate with invalid value

OK (3 tests, 4 assertions)
```

Então é isto, até a próxima!