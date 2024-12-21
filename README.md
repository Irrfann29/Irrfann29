

<h2> Hey there! I'm Irfan</h2>

<h3> 👨🏻‍💻 &nbsp;About Me </h3>

- 🤔 &nbsp; Exploring new technologies and developing software solutions and quick hacks.
- 🎓 &nbsp; Studying Computer Science and Mathematics.
- 🌱 &nbsp; Learning more about Web development,Artificial Intelligence and Machine Learning.


<h3> 🛠 &nbsp;Tech Stack</h3>

- 💻 &nbsp;
  ![Python](https://img.shields.io/badge/-Python-333333?style=flat&logo=python)
  ![Javascript](https://img.shields.io/badge/-Javascript-333333?style=flat&logo=Java&logoColor=007396)
  
- 🌐 &nbsp;
  ![HTML5](https://img.shields.io/badge/-HTML5-333333?style=flat&logo=HTML5)
  ![CSS](https://img.shields.io/badge/-CSS-333333?style=flat&logo=CSS3&logoColor=1572B6)
  ![JavaScript](https://img.shields.io/badge/-JavaScript-333333?style=flat&logo=javascript)
  ![Tailwind](https://img.shields.io/badge/-Tailwind-333333?style=flat&logo=bootstrap&logoColor=563D7C)
  ![Node.js](https://img.shields.io/badge/-Node.js-333333?style=flat&logo=node.js)
  ![React](https://img.shields.io/badge/-React-333333?style=flat&logo=react)
- 🛢 &nbsp;
  ![MySQL](https://img.shields.io/badge/-MySQL-333333?style=flat&logo=mysql)
- ⚙️ &nbsp;
  ![Git](https://img.shields.io/badge/-Git-333333?style=flat&logo=git)
  ![GitHub](https://img.shields.io/badge/-GitHub-333333?style=flat&logo=github)
- 🔧 &nbsp;
  ![Visual Studio Code](https://img.shields.io/badge/-Visual%20Studio%20Code-333333?style=flat&logo=visual-studio-code&logoColor=007ACC)
<br/>

<br/>

<h3> 🤝🏻 &nbsp;Connect with Me </h3>

<p align="center">
  <a href="https://www.linkedin.com/in/irfan-hashmi-86696925b/"><img alt="LinkedIn" src="https://img.shields.io/badge/LinkedIn-Irfan%20Hashmi-blue?style=flat-square&logo=linkedin"></a>
  <a href="mailto:avsingh@umass.edu"><img alt="Email" src="https://img.shields.io/badge/Email-mrirfanhashmi2303@gmail.com-blue?style=flat-square&logo=gmail"></a>
</p>

⭐️ From [Irfan Hashmi](https://github.com/Irrfann29)
<?php

/*
 * This file is part of GitHub Profile Views Counter.
 *
 * (c) Anton Komarev <anton@komarev.com>
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 */

declare(strict_types=1);

use Dotenv\Exception\InvalidPathException;
use Komarev\GitHubProfileViewsCounter\BadgeImageRendererService;
use Komarev\GitHubProfileViewsCounter\CounterRepositoryFactory;
use Komarev\GitHubProfileViewsCounter\Request;
use Komarev\GitHubProfileViewsCounter\Username;
use Komarev\GitHubProfileViewsCounter\Count;

$appBasePath = realpath(__DIR__ . '/..');

require $appBasePath . '/vendor/autoload.php';

$request = Request::of($_SERVER, $_GET);

$username = trim($request->username());

if ($username === '') {
    header('Location: https://github.com/antonkomarev/github-profile-views-counter');
    exit;
}

$isGitHubUserAgent = strpos($request->userAgent(), 'github-camo') === 0;

$badgeLabel = $request->badgeLabel() ?? 'Profile views';
$badgeMessageBackgroundFill = $request->badgeColor() ?? 'blue';
$baseCount = $request->baseCount() ?? '0';
$isCountAbbreviated = $request->isCountAbbreviated();
$badgeStyle = $request->badgeStyle() ?? 'flat';
if (!in_array($badgeStyle, ['flat', 'flat-square', 'plastic', 'for-the-badge', 'pixel'], true)) {
    $badgeStyle = 'flat';
}

header('Content-Type: image/svg+xml');
header('Cache-Control: max-age=0, no-cache, no-store, must-revalidate');

$badgeImageRenderer = new BadgeImageRendererService();

try {
    $counterRepository = (new CounterRepositoryFactory())->create($appBasePath);

    $username = new Username($username);

    if ($isGitHubUserAgent) {
        $counterRepository->addViewByUsername($username);
    }

    if ($badgeStyle === 'pixel') {
        echo $badgeImageRenderer->renderPixel();
    } else {
        $count = new Count(
            $counterRepository->getViewsCountByUsername($username)
        );
        if ($baseCount !== '0') {
            $count = $count->plus(
                Count::ofString($baseCount),
            );
        }

        echo $badgeImageRenderer->renderBadgeWithCount(
            $badgeLabel,
            $count,
            $badgeMessageBackgroundFill,
            $badgeStyle,
            $isCountAbbreviated,
        );
    }
} catch (InvalidPathException $exception) {
    echo $badgeImageRenderer->renderBadgeWithError(
        $badgeLabel,
        'Application environment file is missing',
        $badgeStyle,
    );
} catch (Exception $exception) {
    echo $badgeImageRenderer->renderBadgeWithError(
        $badgeLabel,
        $exception->getMessage(),
        $badgeStyle,
    );
}


