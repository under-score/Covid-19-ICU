## RKI

```R
# tmp = tempfile()
# URL <- c("https://opendata.arcgis.com/datasets/dd4580c810204019a7b8eb3e0b329dd6_0.csv")
# download.file(URL, destfile=tmp, mode='wb')
# rki <- read.csv(tmp,stringsAsFactors =FALSE)
# rki <- rki %>%
#   mutate_if(is.factor, as.character),
#     KreisID = sprintf("%05d", as.numeric(IdLandkreis)),
#     dt= as.Date(Refdatum) ) %>%
#   filter(AnzahlFall>=0) %>%
#   select(KreisID,dt,AnzahlFall,AnzahlTodesfall)

fn <- paste0("/Users/me/Desktop/rki.Rdata")
load(file=fn)
rki <- rki %>%
  group_by(dt, KreisID) %>%
  summarise_at(c("AnzahlFall","AnzahlTodesfall"),  sum) %>%
  mutate(TF14 = slider::slide_dbl(AnzahlTodesfall, sum, .after = 14, .complete = TRUE) ) %>%
  mutate(AF14  = slider::slide_dbl(AnzahlFall,     sum, .before = 14, .complete = TRUE) ) %>%
  mutate(CFR = TF14/AF14*100)
```

## DIVI

```R
# open -a "Google Chrome" https://www.datawrapper.de/_/wwQvR/
# daily task ...
# dk <- c( "2020-11-01.csv","2020-11-04.csv","2020-11-05.csv","2020-11-06.csv","2020-11-07.csv","2020-11-08.csv","2020-11-10.csv","2020-11-11.csv","2020-11-13.csv","2020-11-14.csv","2020-11-15.csv","2020-11-16.csv","2020-11-18.csv","2020-11-20.csv","2020-11-22.csv","2020-11-24.csv","2020-11-26.csv","2020-11-28.csv","2020-11-30.csv")
# fn=paste0("/Users/me/Desktop/",dk[1])
# divi <-read.csv(fn,stringsAsFactors =FALSE)
# divi$dt <- c(dk[1])
# for (i in dk[-1]) {
#  fn=paste0("/Users/me/Desktop/",i)
#  tmp <-read.csv(fn,stringsAsFactors =FALSE) %>%
#    mutate(dt=i) %>%
#    rename_with(~ gsub(".", "PERCENT", .x, fixed = TRUE))
#  divi <- rbind(divi,tmp)
# }
# divi <- divi %>%
#  mutate( KreisID = sprintf("%05d", as.numeric(ID)) ) %>%
#  mutate( dt= as.Date( gsub(".csv", "PERCENT", dt, fixed = TRUE)) ) %>%
#  select(KreisID,dt, COVID, COVIDBEATMET)

fn <- paste0("/Users/me/Desktop/divi.Rdata")
load(file=fn)
```

## Figure

```R
divi %>%
  left_join(rki, by=c("dt","KreisID")) %>%
  group_by(dt, KreisID) %>%
  mutate( t1= COVIDBEATMET/(COVID+COVIDBEATMET)) %>%
  mutate( t2= COVIDBEATMET/AF14 ) %>%
  filter(dt>as.Date("2020-10-31") & dt<=as.Date("2020-12-1")) %>%
  ggplot( aes(x=dt,y=CFR )) +
  # ggplot( aes(x=t1,y=CFR )) +
  # ggplot( aes(x=t2,y=CFR )) +
    geom_smooth( ) +
    coord_cartesian( xlim = NULL, ylim = c(1,3.5) )
```
    
    
    
    
    
    

