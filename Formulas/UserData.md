## Weighted Score
Across multiple tables, Format as example:
```gs
=IF(
  OR(LEN(B2)=0, NOT(ISNUMBER(G2)), NOT(ISNUMBER(H2)), NOT(ISNUMBER(I2))),
  "",
  LET(
    targetCount, G2,
    targetHours, H2,
    targetMean,  I2,

    globalMean, AVERAGE(FormatDataLocal[User Mean Score]),
    kc, PERCENTILE.INC(FormatDataLocal[User Count], 0.5),
    kh, PERCENTILE.INC(FormatDataLocal[User Hours], 0.5),

    cEff, IF(kc<=0, 0, targetCount / (targetCount + kc)),
    hEff, IF(kh<=0, 0, targetHours / (targetHours + kh)),
    confidence_raw, 0.5*cEff + 0.5*hEff,

    smallCap, 0.35,
    confidence, MAX( MIN(confidence_raw, 0.95), smallCap*(cEff+hEff>0) ),

    weightedScore, globalMean + (targetMean - globalMean) * confidence,
    ROUND(weightedScore, 4)
  )
)
```

## Normalized Score
Across multiple tables, Format as example:
```gs
=IF(OR(G2=0, LEN(G2)=0),
   "",
   LET(
     w,   J2, 
     nums, FILTER(FormatDataLocal[User Weighted Score], ISNUMBER(FormatDataLocal[User Weighted Score])),
     minW, MIN(nums),
     maxW, MAX(nums),
     top, Settings!$P$20,
     bot, Settings!$P$21,
     mid, (top+bot)/2,
     avgW, AVERAGE(nums),
     scaled,
       IF(
         w=avgW,
         mid,
         IF(
           w>avgW,
           mid + (w-avgW)/(maxW-avgW) * (top-mid),
           mid - (avgW-w)/(avgW-minW) * (mid-bot)
         )
       ),
     ROUND(scaled,2)
   )
)
```
