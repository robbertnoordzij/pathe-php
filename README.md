Pathe PHP client
================

[![Build Status](https://travis-ci.org/rickdenhaan/pathe-php.png?branch=master)](https://travis-ci.org/rickdenhaan/pathe-php)
[![Coverage Status](https://coveralls.io/repos/rickdenhaan/pathe-php/badge.png?branch=master)](https://coveralls.io/r/rickdenhaan/pathe-php)

This PHP client connects to the Pathé Client Panel at [onlinetickets2.pathe.nl](https://onlinetickets2.pathe.nl/ticketweb.php?sign=30&UserCenterID=1) to retrieve information about an Unlimited or Gold Member's history.

Many thanks to [Robbert Noordzij](https://github.com/robbertnoordzij) for his efforts in getting this project started!


Usage
-----

```php
use Capirussa\Pathe;

try {
    $client = new Pathe\Client('username', 'password');

    $client->register(new Pathe\PersonalData()); // note: you really do need to fill this PersonalData object first!

    $customerDetails    = $client->getPersonalData();
    $movieHistory       = $client->getCustomerHistory();
    $reservationHistory = $client->getReservationHistory();
    $cardHistory        = $client->getCardHistory('cardNumber', 'pinCode');

    $customerDetails->setNewsletter(true);
    $customerDetails->setPassword('newPassword123');

    $updatedCustomerDetails = $client->updatePersonalData($customerDetails);

    $success = $client->forgotPassword('email@example.com');
    $success = $client->deleteAccount();
} catch (\Exception $exception) {
    // something went wrong, fix it and try again!
}
```


Pathe\Client
------------

You need to supply your Mijn Pathé username and password when initializing the Pathe\Client.

On some servers, usually localhost development servers, you might encounter SSL errors. To skip the SSL validation, call $client->disableSslVerification().

There are several pieces of information you can retrieve from Pathé:

* Your personal details. Calling $client->getPersonalData() will return a Pathe\PersonalData object.
* The last 100 movies you watched. Calling $client->getCustomerHistory() will return an array of Pathe\HistoryItem entities
* Your reservation history. Calling $client->getReservationHistory() will return an array of Pathe\HistoryItem entities
* Your Unlimited/Gold Card usage history. Calling $client->getCardHistory() with a card number and PIN code will return an array of Pathe\HistoryItem entities, of which the Reservation contains the date/time at which the tickets were picked up. It is possible to filter the result by month, by supplying the month as a third argument and the year as a fourth argument.

It is also possible to update your Pathé user details by calling $client->updatePersonalData() with a modified Pathe\PersonalData object. This will return a new Pathe\PersonalData object, freshly retrieved from Pathé after sending the updated information.

If you've forgotten your password, you can also request a reset link by calling $client->forgotPassword() with your email address. This will return a boolean true or false, depending on whether the request was successful.

If you do not have an account yet, you can now call $client->registerAccount() with a (filled) PersonalData object to register for a new account. Unfortunately, it is not possible to also request an Unlimited or Gold membership with Pathé, you will need to physically visit one of the theaters to do that.

If you do not have an Unlimited or Gold card, it is also possible to delete your account by calling $client->deleteAccount(). This will return a boolean true or false, depending on whether or not your account was successfully deleted.

More options will be added soon!


Pathe\PersonalData
------------------

The Pathe\PersonalData entity contains your Pathé customer details:

* Username - a string containing your username (not actually fetched from Pathé, but inserted into this object from your Pathe\Client configuration
* Password - does *not* contain your current password, but is used for *changing* your password. Contains Pathe\PersonalData::PASSWORD_NO_CHANGE until you change it
* Gender - a numeric key to indicate your gender, matches either Pathe\PersonalData::GENDER_MALE or Pathe\PersonalData::GENDER_FEMALE
* FirstName - a string containing your first name
* MiddleName - a string containing the prefix for your last name (e.g. "van", "de", "van der", etc.) or null if your name does not have one
* LastName - a string containing your last name
* EmailAddress - a string containing your email address
* Country - a numeric identifier for your country, for now only matches Pathe\PersonalData::COUNTRY_NETHERLANDS
* BirthDate - a DateTime object containing your birth date
* StreetName - a string containing your street
* HouseNumber - an integer containing your house number
* HouseNumberSuffix - an optional string that is part of your house number, for example if you live on number 1a, HouseNumber will be 1 and HouseNumberSuffix will be "a"
* PostalCode - the postal code of your address
* City - your city
* MobilePhoneNumber - a 10-digit string of your mobile phone number, or null if not known
* Newsletter - a boolean indicating whether or not you want to receive the weekly Pathé newsletters


Pathe\HistoryItem
-----------------

The Pathe\HistoryItem entity contains three properties:

* ShowTime - a DateTime object containing the date and time at which the movie started
* Screen - a Screen object containing the theater and screen at which the movie played
* Event - an Event object containing the name of the movie that you watched
* Reservation - a Reservation object containing details of the reservation (only if fetched via $client->getReservationHistory())


Pathe\Screen
------------

The Pathe\Screen entity contains two properties:

* Theater - the name of the theater at which the movie played
* Screen - the screen on which the movie played


Pathe\Event
-----------

The Pathe\Event entity for now only contains the name of the movie you watched:

* MovieName - the name of the movie that you watched


Pathe\Reservation
-----------------

The Pathe\Reservation entity contains details about a past reservation:

* TicketCount - the number of tickets in this reservation
* Status - the status of this reservation, can be one of Pathe\Reservation::STATUS_UNKNOWN, Pathe\Reservation::STATUS_COLLECTED or Pathe\Reservation::STATUS_DELETED
* ShowIdentifier - not sure yet what this is, but I think it's a unique identifier for the movie
* ReservationSetIdentifier - appears to be the unique reservation identifier
* CollectionNumber - not sure what this is used for, but we might need it in the future
* CollectDateTime - the date/time at which the tickets for this reservation were collected

Note that the Reservation object has not yet been tested against active reservations (which have not yet been collected and have not been deleted) so if you have one of those, calling $client->getReservationHistory() will throw an InvalidArgumentException saying the status is invalid. This will be fixed soon.


If you find any bugs or have any feature requests, please raise an issue on Github or fork this project and create a pull request.

Happy coding!
