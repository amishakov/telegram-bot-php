# PHP Telegram Bot

Telegram bot as it should be.

## Examples

### SetWebhook

```php
<?php
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\SetWebhook;
use function GuzzleHttp\Psr7\str;

$bot = new Bot('telegramToken');

$setWebhook = new SetWebhook('https://your.app/telegram-updates-hook');

// uncomment if use use self-signed certificate
// $setWebhook->setCertificate(file_get_contents('/path/to/self-signed-certifcate.pem'));

$response = $bot->setWebhook($setWebhook);

echo str($response);
```

### GetWebhookInfo

```php
<?php
use Formapro\TelegramBot\Bot;
use function GuzzleHttp\Psr7\str;

$bot = new Bot('telegramToken');

$response = $bot->getWebhookInfo();

echo str($response);
```

### SendMessage

```php
<?php
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\Update;
use Formapro\TelegramBot\SendMessage;

$requestBody = file_get_contents('php://input'); 
$data = json_decode($requestBody, true);

$update = Update::create($data);

$bot = new Bot('telegramToken');
$bot->sendMessage(new SendMessage(
    $update->getMessage()->getChat()->getId(),
    'Hi there! What can I do?'
));
```

### SendPhoto

**Note:** if you want to send gif with many pictures inside - 
use `sendDocument`

```php
<?php
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\Update;
use Formapro\TelegramBot\SendMessage;

$requestBody = file_get_contents('php://input');
$data = json_decode($requestBody, true);

$update = Update::create($data);

// You can pass URI of image or a path to file
$picture = '/path/to/picture'; // OR https://some-server.com/some.jpg
 
$sendPhoto = new SendPhoto(
    $update->getMessage()->getChat()->getId(),
    file_get_contents($picture) // or just $picture if it's url
);

// also you can set `caption` to image
$sendPhoto->setCaption('Some caption under the picture');

$bot = new Bot('telegramToken');
$bot->sendPhoto($sendPhoto);
```

### SendDocument

**Note:** _sending by URL will currently only work for
gif, pdf and zip files._ (from documentation)

```php
<?php
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\Update;
use Formapro\TelegramBot\SendDocument;
use Formapro\TelegramBot\FileUrl;
use Formapro\TelegramBot\FileId;
use Formapro\TelegramBot\InputFile;

$requestBody = file_get_contents('php://input');
$data = json_decode($requestBody, true);

$update = Update::create($data);

// You can pass URI of image or a path to file
//$document = new FileUrl('https://some-server.com/some.pdf');

// You can pass an ID of already stored file on Telegram side.
//$document = new FileId('123');

// You can pass local file.
$document = new InputFile('test.txt', 'Hi there!');

$sendDocument = SendDocument::withInputFile(
    $update->getMessage()->getChat()->getId(),
    $document
);

// also you can set `caption` to image
$sendDocument->setCaption('Some caption under the document');

$bot = new Bot('telegramToken');
$bot->sendDocument($sendDocument);
```

### SendInvoice

```php
<?php
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\Update;
use Formapro\TelegramBot\SendInvoice;

$requestBody = file_get_contents('php://input');
$data = json_decode($requestBody, true);

$update = Update::create($data);
$payload = []; // any params which you need to proccess in the transaction
$providerToken = 'something:like:this'; // Token have to be taken from botFather

$sendInvoice = new SendInvoice(
    $update->getMessage()->getChat()->getId(),
    'Your title',
    'Your description of invoice',
    json_encode($payload), 
    $providerToken,
    '12314czasdq', // unique id
    'UAH',
    [new LabeledPrice('PriceLabel_1', 3001)] // amount; here 30.01 UAH
);

$bot = new Bot('telegramToken');
$bot->sendInvoice($sendInvoice);
```

### ReplyKeyboardMarkup

```php
<?php
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\Update;
use Formapro\TelegramBot\SendMessage;
use Formapro\TelegramBot\KeyboardButton;
use Formapro\TelegramBot\ReplyKeyboardMarkup;

$requestBody = file_get_contents('php://input'); 
$data = json_decode($requestBody, true);

$update = Update::create($data);

$fooButton = new KeyboardButton('foo');
$barButton = new KeyboardButton('bar');
$bazButton = new KeyboardButton('bar');
$keyboard = new ReplyKeyboardMarkup([[$fooButton], [$barButton, $bazButton]]);

$sendMessage = new SendMessage($update->getMessage()->getChat()->getId(), 'Choose an option.');
$sendMessage->setReplyMarkup($keyboard);

$bot = new Bot('telegramToken');
$bot->sendMessage($sendMessage);
```

Request Contacts:

```php
<?php
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\Update;
use Formapro\TelegramBot\SendMessage;
use Formapro\TelegramBot\KeyboardButton;
use Formapro\TelegramBot\ReplyKeyboardMarkup;

$requestBody = file_get_contents('php://input'); 
$data = json_decode($requestBody, true);

$update = Update::create($data);

$button = new KeyboardButton('Share my contacts');
$button->setRequestContact(true);
$keyboard = new ReplyKeyboardMarkup([[$button]]);
$keyboard->setOneTimeKeyboard(true);

$sendMessage = new SendMessage($update->getMessage()->getChat()->getId(), 'Please, share your contact info with us.');
$sendMessage->setReplyMarkup($keyboard);

$bot = new Bot('telegramToken');
$bot->sendMessage($sendMessage);
```

### InlineKeyboardButton

Url: 

```php
<?php
use Formapro\TelegramBot\InlineKeyboardButton;
use Formapro\TelegramBot\InlineKeyboardMarkup;
use Formapro\TelegramBot\SendMessage;
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\Update;

$requestBody = file_get_contents('php://input'); 
$data = json_decode($requestBody, true);

$update = Update::create($data);

$button = InlineKeyboardButton::withUrl('inline button', 'https://your.app/link');
$keyboard = new InlineKeyboardMarkup([[$button]]);
        
$sendMessage = new SendMessage($update->getMessage()->getChat()->getId(), 'Click on inline button.');
$sendMessage->setReplyMarkup($keyboard);
                
$bot = new Bot('telegramToken');
$bot->sendMessage($sendMessage);
```

CallbackQuery: 

```php
<?php
use Formapro\TelegramBot\InlineKeyboardButton;
use Formapro\TelegramBot\InlineKeyboardMarkup;
use Formapro\TelegramBot\SendMessage;
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\Update;

$requestBody = file_get_contents('php://input'); 
$data = json_decode($requestBody, true);

$update = Update::create($data);

$button = InlineKeyboardButton::withCallbackData('inline button', 'some_data');
$keyboard = new InlineKeyboardMarkup([[$button]]);
        
$sendMessage = new SendMessage($update->getMessage()->getChat()->getId(), 'Click on inline button.');
$sendMessage->setReplyMarkup($keyboard);
                
$bot = new Bot('telegramToken');
$bot->sendMessage($sendMessage);
```

## AnswerCallbackQuery

```php
<?php
use Formapro\TelegramBot\AnswerCallbackQuery;
use Formapro\TelegramBot\Bot;
use Formapro\TelegramBot\Update;

$requestBody = file_get_contents('php://input'); 
$data = json_decode($requestBody, true);

$update = Update::create($data);

if ($callbackQuery = $update->getCallbackQuery()) {
    $bot = new Bot('telegramToken');
    $bot->answerCallbackQuery(new AnswerCallbackQuery($callbackQuery->getId()));
}
```

## Developed by Forma Pro

Forma Pro is a senior product engineering company helping product organizations build and evolve complex, data-intensive software.

Our public open-source work reflects more than two decades of engineering history, including PHP/Symfony libraries, infrastructure tooling, workflow components, and messaging systems.

Today, our work spans marketplaces and platforms, partner and attribution systems, FinTech, logistics, EdTech, data-intensive workflows, matching and recommendation systems, and applied AI/ML.

Learn more at [forma-pro.com](https://forma-pro.com).

For questions about this project or our open-source work, contact us at opensource@forma-pro.com.

## License

It is released under the [MIT License](LICENSE).
