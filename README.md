# LulzCapita investment tracker server

Stock portfolio exporter and analysis tool for people not afraid to
show what they've got.

More information coming later on...

## Database schema

The database document structure is defined below. The schema depends
on <tt>table</tt> field.

Possibly sensitive fields like transaction ID, bank ID and account ID
are hashed. The hash function is "half-SHA256", meaning that normal
SHA256 is calculated but only first half of its internal state is
returned. Initial value (so-called salt) is server dependant and is
defined in
<tt>[sink.conf](https://github.com/zouppen/lulzcapita/blob/master/sink.conf.example#L7)</tt>. More
about hash generation at
[Common.hs](https://github.com/zouppen/lulzcapita/blob/master/Common.hs#L48)
function <tt>hashGen</tt>.

### portfolio

* <tt>\_id</tt>: Transaction ID, hashed with bank id and account ID
* <tt>user</tt>: User ID
* <tt>date</tt>: Date of transaction
* <tt>type</tt>: Type of transaction. One of the following:
  * <tt>account</tt>: Deposit or withdraw, depending of the sign of sum
  * <tt>tax</tt>: Taxation.
  * <tt>sale</tt>: Purchase or sale, depending of the sign of count
  * <tt>income</tt>: Return of capital, dividents, interests
* <tt>count</tt>: In case of sales, this contains the number of stock or 
  security sold. In case of purcase, this is positive and in case of sale
  this is negative.
* <tt>isin</tt>: The ISIN of the stock in question.

In database, taxation on account interest, dividents and sales have
the type of "tax". In case of account taxation, "isin" field is null,
otherwise it contains the ISIN of that stock or security that has been
taxated.

### user

* <tt>\_id</tt>: User ID, autogenerated by CouchDB
* <tt>nick</tt>: Nickname
* <tt>portfolios</tt>: An array of portfolio IDs belonging to this user.
  ID's are hashed with bank ID.

### sync_info

* <tt>\_id</tt>: Portfolio ID, hashed with bank ID.
* <tt>date</tt>: Date of last successful sync.

## Server configuration

TODO nginx instructions

## Installation

These instructions are for Debian and Ubuntu. Feel free to send a pull
request containing installation instructions for other operating
systems.

Install some packages:

    sudo apt-get install spawn-fcgi haskell-platform libghc6-fastcgi-dev

Then, compile the binary:

    ghc --make -o sink Sink.hs

## Running

To run interactively as a FastCGI on port 9001.

    spawn-fcgi -n -p 9001 -- ./sink
