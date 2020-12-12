<!-- markdownlint-disable MD002 MD041 -->

In diesem Lernprogramm erfahren Sie, wie Sie eine Ruby on Rails-Webanwendung erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.

> [!TIP]
> Wenn Sie das fertige Lernprogramm einfach herunterladen möchten, können Sie es auf zwei Arten herunterladen.
>
> - Laden Sie den [Ruby-Schnellstart](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) herunter, um in wenigen Minuten Arbeitscode zu erhalten.
> - Laden Sie das [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)herunter, oder Klonen Sie es.

## <a name="prerequisites"></a>Voraussetzungen

Bevor Sie dieses Lernprogramm starten, sollten Sie die folgenden Tools auf dem Entwicklungscomputer installiert haben.

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [Sqlite3](https://sqlite.org/index.html)
- [Node.js](https://nodejs.org/en/)
- [Garn](https://yarnpkg.com/)

Sie sollten auch über ein persönliches Microsoft-Konto mit einem Postfach auf Outlook.com oder ein Microsoft-Arbeits-oder Schulkonto verfügen. Wenn Sie kein Microsoft-Konto haben, gibt es mehrere Optionen, um ein kostenloses Konto zu erhalten:

- Sie können [sich für ein neues persönliches Microsoft-Konto registrieren](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).
- Sie können sich [für das Office 365 Entwicklerprogramm registrieren](https://developer.microsoft.com/office/dev-program) , um ein kostenloses Office 365-Abonnement zu erhalten.

> [!NOTE]
> Dieses Lernprogramm wurde mit den folgenden Versionen der erforderlichen Tools geschrieben. Die Schritte in diesem Leitfaden funktionieren möglicherweise mit anderen Versionen, jedoch nicht getestet.
>
> - Ruby-Version 2.6.6.
> - Sqlite3 Version 3.31.1
> - Node.js Version 14.15.0
> - Garn Version 1.22.0

## <a name="feedback"></a>Feedback

Geben Sie Feedback zu diesem Lernprogramm im [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)an.
