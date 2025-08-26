## RecPool Formula
```gs
=IFERROR(
  LET(
    topN,         $D$3,
    onlyDub,      $D$4,
    removeAiring, $D$5,
    useDC,        $D$6,
    dcThresh,     $D$7,

    fmtSet,     IF(LEN($D$8),  TRIM(SPLIT(SUBSTITUTE($D$8, ",", "|"), "|")),  ),
    genSet,     IF(LEN($D$9),  TRIM(SPLIT(SUBSTITUTE($D$9, ",", "|"), "|")),  ),
    themeSet,   IF(LEN($D$10), TRIM(SPLIT(SUBSTITUTE($D$10,",", "|"), "|")),  ),
    demoSet,    IF(LEN($D$11), TRIM(SPLIT(SUBSTITUTE($D$11,",", "|"), "|")),  ),
    countrySet, IF(LEN($D$12), TRIM(SPLIT(SUBSTITUTE($D$12,",", "|"), "|")),  ),

    genAlt,    IF(LEN($D$9),  TEXTJOIN("|", TRUE, genSet),   ""),
    themeAlt,  IF(LEN($D$10), TEXTJOIN("|", TRUE, themeSet), ""),
    demoAlt,   IF(LEN($D$11), TEXTJOIN("|", TRUE, demoSet),  ""),

    out,
    {
      AnimeDataLocal[Anime ID],
      AnimeDataLocal[English Name],
      AnimeDataLocal[Generated Rank],
      AnimeDataLocal[Generated Score],

      BYROW(
        AnimeDataLocal[Country ID],
        LAMBDA(cid,
          IF(LEN(cid),
            XLOOKUP(cid, CountryDataLocal[ID], CountryDataLocal[Icon], ""),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[Studio ID],
        LAMBDA(csv,
          IF(LEN(csv),
            LET(
              toks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv, " ", ""), ",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv, " ", ""), ",")))),
              names, MAP(toks, LAMBDA(t, IFERROR(INDEX(StudioDataLocal[Name], MATCH(t, StudioDataLocal[ID], 0)), ""))),
              TEXTJOIN(", ", TRUE, names)
            ),
            ""
          )
        )
      ),
      
      BYROW(
        AnimeDataLocal[Genre ID],
        LAMBDA(csv,
          IF(LEN(csv),
            LET(
              toks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv, " ", ""), ",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv, " ", ""), ",")))),
              names, MAP(toks, LAMBDA(t, IFERROR(INDEX(GenreDataLocal[Name], MATCH(t, GenreDataLocal[ID], 0)), ""))),
              TEXTJOIN(", ", TRUE, names)
            ),
            ""
          )
        )
      ),
      
      BYROW(
        AnimeDataLocal[Theme ID],
        LAMBDA(csv,
          IF(LEN(csv),
            LET(
              toks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv, " ", ""), ",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv, " ", ""), ",")))),
              names, MAP(toks, LAMBDA(t, IFERROR(INDEX(ThemeDataLocal[Name], MATCH(t, ThemeDataLocal[ID], 0)), ""))),
              TEXTJOIN(", ", TRUE, names)
            ),
            ""
          )
        )
      ),
      
      BYROW(
        AnimeDataLocal[Demographic ID],
        LAMBDA(csv,
          IF(LEN(csv),
            LET(
              toks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv, " ", ""), ",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv, " ", ""), ",")))),
              names, MAP(toks, LAMBDA(t, IFERROR(INDEX(DemographicDataLocal[Name], MATCH(t, DemographicDataLocal[ID], 0)), ""))),
              TEXTJOIN(", ", TRUE, names)
            ),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[Franchise ID],
        LAMBDA(fid,
          IF(LEN(fid),
            XLOOKUP(fid, FranchiseDataLocal[ID], FranchiseDataLocal[Name], ""),
            ""
          )
        )
      ),

      AnimeDataLocal[Relation],

      BYROW(
        AnimeDataLocal[AniDB ID],
        LAMBDA(id,
          IF(LEN(id),
            HYPERLINK("https://anidb.net/anime/"&id, IMAGE("https://anidb.net/favicon.ico",4,16,16)),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[AniList ID],
        LAMBDA(id,
          IF(LEN(id),
            HYPERLINK("https://anilist.co/anime/"&id, IMAGE("https://anilist.co/img/icons/favicon-32x32.png",4,16,16)),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[MAL ID],
        LAMBDA(id,
          IF(LEN(id),
            HYPERLINK("https://myanimelist.net/anime/"&id, IMAGE("https://myanimelist.net/favicon.ico",4,16,16)),
            ""
          )
        )
      ),

      AnimeDataLocal[Data Completion]
    },

    filtered,
    FILTER(
      out,

      ISNUMBER(AnimeDataLocal[Generated Rank]),
      AnimeDataLocal[Status]="Unknown",
      AnimeDataLocal[Watchable]=TRUE,

      IF(onlyDub,      AnimeDataLocal[Dubbed]<>FALSE,                 ROW(AnimeDataLocal[Anime ID])>0),
      IF(removeAiring, NOT(AnimeDataLocal[Airing]),                   ROW(AnimeDataLocal[Anime ID])>0),
      IF(useDC,        AnimeDataLocal[Data Completion] >= dcThresh,   ROW(AnimeDataLocal[Anime ID])>0),

      IF(
        LEN($D$8),
        NOT( ISNUMBER( MATCH( AnimeDataLocal[Format ID], TRIM(SPLIT($D$8, ",")), 0 ) ) ),
        ROW(AnimeDataLocal[Anime ID])>0
      ),
      
      IF(
        LEN($D$9),
        NOT(
          BYROW(
            AnimeDataLocal[Genre ID],
            LAMBDA(csv,
              LET(
                rowToks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")))),
                inSet,    TRIM(SPLIT($D$9, ",")),
                anyHit,   SUM( N( ISNUMBER( MATCH(rowToks, inSet, 0) ) ) ) > 0,
                anyHit
              )
            )
          )
        ),
        ROW(AnimeDataLocal[Anime ID])>0
      ),
      
      IF(
        LEN($D$10),
        NOT(
          BYROW(
            AnimeDataLocal[Theme ID],
            LAMBDA(csv,
              LET(
                rowToks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")))),
                inSet,    TRIM(SPLIT($D$10, ",")),
                anyHit,   SUM( N( ISNUMBER( MATCH(rowToks, inSet, 0) ) ) ) > 0,
                anyHit
              )
            )
          )
        ),
        ROW(AnimeDataLocal[Anime ID])>0
      ),
      
      IF(
        LEN($D$11),
        NOT(
          BYROW(
            AnimeDataLocal[Demographic ID],
            LAMBDA(csv,
              LET(
                rowToks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")))),
                inSet,    TRIM(SPLIT($D$11, ",")),
                anyHit,   SUM( N( ISNUMBER( MATCH(rowToks, inSet, 0) ) ) ) > 0,
                anyHit
              )
            )
          )
        ),
        ROW(AnimeDataLocal[Anime ID])>0
      ),
      
      IF(
        LEN($D$12),
        NOT( ISNUMBER( MATCH( AnimeDataLocal[Country ID], TRIM(SPLIT($D$12, ",")), 0 ) ) ),
        ROW(AnimeDataLocal[Anime ID])>0
      )
    ),
    SORTN(filtered, topN, TRUE, 4, FALSE)
  ),
  ""
)
```

## WatchNext Fomrula
```gs
=IFERROR(
  LET(
    topN,         $D$3,
    onlyDub,      $D$4,
    removeAiring, $D$5,
    useDC,        $D$6,
    dcThresh,     $D$7,

    out,
    {
      AnimeDataLocal[Anime ID],
      AnimeDataLocal[English Name],
      AnimeDataLocal[WatchNext Score],
      IFERROR(AnimeDataLocal[User Rank],     AnimeDataLocal[Generated Rank]),  
      IFERROR(AnimeDataLocal[User Score],    AnimeDataLocal[Generated Score]),
      AnimeDataLocal[Status],

      BYROW(
        AnimeDataLocal[Country ID],
        LAMBDA(cid,
          IF(LEN(cid),
            XLOOKUP(cid, CountryDataLocal[ID], CountryDataLocal[Icon], ""),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[Studio ID],
        LAMBDA(csv,
          IF(LEN(csv),
            LET(
              toks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")))),
              names, MAP(toks, LAMBDA(t, IFERROR(INDEX(StudioDataLocal[Name], MATCH(t, StudioDataLocal[ID], 0)), ""))),
              TEXTJOIN(", ", TRUE, names)
            ),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[Genre ID],
        LAMBDA(csv,
          IF(LEN(csv),
            LET(
              toks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")))),
              names, MAP(toks, LAMBDA(t, IFERROR(INDEX(GenreDataLocal[Name], MATCH(t, GenreDataLocal[ID], 0)), ""))),
              TEXTJOIN(", ", TRUE, names)
            ),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[Theme ID],
        LAMBDA(csv,
          IF(LEN(csv),
            LET(
              toks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")))),
              names, MAP(toks, LAMBDA(t, IFERROR(INDEX(ThemeDataLocal[Name], MATCH(t, ThemeDataLocal[ID], 0)), ""))),
              TEXTJOIN(", ", TRUE, names)
            ),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[Demographic ID],
        LAMBDA(csv,
          IF(LEN(csv),
            LET(
              toks,  FILTER(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")), LEN(TRIM(SPLIT(SUBSTITUTE(csv," ",""),",")))),
              names, MAP(toks, LAMBDA(t, IFERROR(INDEX(DemographicDataLocal[Name], MATCH(t, DemographicDataLocal[ID], 0)), ""))),
              TEXTJOIN(", ", TRUE, names)
            ),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[Franchise ID],
        LAMBDA(fid,
          IF(LEN(fid),
            XLOOKUP(fid, FranchiseDataLocal[ID], FranchiseDataLocal[Name], ""),
            ""
          )
        )
      ),

      AnimeDataLocal[Relation],

      BYROW(
        AnimeDataLocal[AniDB ID],
        LAMBDA(id,
          IF(LEN(id),
            HYPERLINK("https://anidb.net/anime/"&id, IMAGE("https://anidb.net/favicon.ico",4,16,16)),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[AniList ID],
        LAMBDA(id,
          IF(LEN(id),
            HYPERLINK("https://anilist.co/anime/"&id, IMAGE("https://anilist.co/img/icons/favicon-32x32.png",4,16,16)),
            ""
          )
        )
      ),

      BYROW(
        AnimeDataLocal[MAL ID],
        LAMBDA(id,
          IF(LEN(id),
            HYPERLINK("https://myanimelist.net/anime/"&id, IMAGE("https://myanimelist.net/favicon.ico",4,16,16)),
            ""
          )
        )
      ),

      AnimeDataLocal[Data Completion]
    },

    filtered,
    FILTER(
      out,

      AnimeDataLocal[Watch Next]=TRUE,

      IF(onlyDub,      AnimeDataLocal[Dubbed]<>FALSE,               ROW(AnimeDataLocal[Anime ID])>0),

      IF(removeAiring, NOT(AnimeDataLocal[Airing]),                 ROW(AnimeDataLocal[Anime ID])>0),

      IF(useDC,        AnimeDataLocal[Data Completion] >= dcThresh, ROW(AnimeDataLocal[Anime ID])>0),

      IF(LEN($D$8),
        NOT( ISNUMBER( MATCH( AnimeDataLocal[Format ID], TRIM(SPLIT(SUBSTITUTE($D$8,",","|"),"|")), 0 ) ) ),
        ROW(AnimeDataLocal[Anime ID])>0
      ),

      IF(LEN($D$9),
        NOT( REGEXMATCH(","&AnimeDataLocal[Genre ID]&",",
                         ",(" & TEXTJOIN("|",TRUE,TRIM(SPLIT(SUBSTITUTE($D$9,",","|"),"|"))) & ")(,|$)") ),
        ROW(AnimeDataLocal[Anime ID])>0
      ),

      IF(LEN($D$10),
        NOT( REGEXMATCH(","&AnimeDataLocal[Theme ID]&",",
                         ",(" & TEXTJOIN("|",TRUE,TRIM(SPLIT(SUBSTITUTE($D$10,",","|"),"|"))) & ")(,|$)") ),
        ROW(AnimeDataLocal[Anime ID])>0
      ),

      IF(LEN($D$11),
        NOT( REGEXMATCH(","&AnimeDataLocal[Demographic ID]&",",
                         ",(" & TEXTJOIN("|",TRUE,TRIM(SPLIT(SUBSTITUTE($D$11,",","|"),"|"))) & ")(,|$)") ),
        ROW(AnimeDataLocal[Anime ID])>0
      ),

      IF(LEN($D$12),
        NOT( ISNUMBER( MATCH( AnimeDataLocal[Country ID], TRIM(SPLIT(SUBSTITUTE($D$12,",","|"),"|")), 0 ) ) ),
        ROW(AnimeDataLocal[Anime ID])>0
      )
    ),

    SORTN(filtered, topN, TRUE, 3, FALSE)
  ),
  ""
)
```
