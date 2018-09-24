# sheets-web-scrape
Scraping, querying, and repackaging web data via Google Sheets

The client needed a more efficient solution for gathering data from web browser-based tables. He had been relying on copy and pasting individual data items or retyping data from a single table. The solution here is part of a larger XML import solution, but shows the possibilities for scraping, running logic, and repackaging data via Google Sheets -- particularly when the data provider does not provide a RESTful service.

The goal in this module, which draws from a real estate listing table, is for the user to:
 1 determine if the listing start date falls within the past year,
 2 and if so, write a short paragraph describing the listing, including listing date, original asking price, current state, and current state asking price.

The user copies an entire web browser table into a tab, in this case named 'mls_dom'.

The entire module is wrapped in  an IF() function, which tests the number of days between TODAY() and 'New Listing' QUERY() result.

If the difference is equal to or less than 365, a CONCATENATE() function triggers with a series of template text strings and QUERY() functions. That is wrapped in a regular expressions function, REGEXREPLACE(), to address dollar sign escape issue in a parent module.

'''
=
IF(
  365 >= DAYS(
    TODAY(),
    QUERY(
      mls_dom!A2:I,
      "SELECT B WHERE F = 'New Listing' LIMIT 1",
      0
    )
  ),
  REGEXREPLACE(
    CONCATENATE(
      "NEOHREX MLS# ",
      QUERY(
        mls_dom!A2:I,
        "SELECT A WHERE F = 'New Listing' LIMIT 1",
        0
      ),
      " listed ",
      CONCATENATE(
        REGEXEXTRACT(
          QUERY(
            mls_dom!A2:I,
            "SELECT B WHERE F = 'New Listing' LIMIT 1",
            0
          ),
          "(\d\d/\d\d/)1\d"
        ),
        YEAR(
          REGEXEXTRACT(
            QUERY(
              mls_dom!A2:I,
              "SELECT B WHERE F = 'New Listing' LIMIT 1",
              0
            ),
            "\d\d/\d\d/\d\d"
          )
        ),
        " at ",
        QUERY(
          mls_dom!A2:I,
          "SELECT C WHERE F = 'New Listing' LIMIT 1",
          0
        ),
        ". ",
        "Currently shows ",
        QUERY(
          mls_dom!A2:I,
          "SELECT F WHERE A = '"&
            QUERY(
              mls_dom!A2:I,
              "SELECT A WHERE F = 'New Listing' LIMIT 1",
              0
            )
            &"' LIMIT 1",
          0
        ),
        " at ",
        QUERY(
          mls_dom!A2:I,
          "SELECT C WHERE A = '"&
            QUERY(
              mls_dom!A2:I,
              "SELECT A WHERE F = 'New Listing' LIMIT 1",
              0
            )
          &"' LIMIT 1",
          0
        ),
        "."
      )
    ),
    "\$",
    "\\$"
  ),
  "No MLS listings within the past year."
)
'''
