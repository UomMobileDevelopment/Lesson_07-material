# Lesson 07

## Οδηγίες ασφαλούς χρήσης του Open Maps Weather API key.

Όπως είπαμε και σήμερα στο μάθημα, ΔΕΝ πρέπει να δημοσιεύσετε το κλειδί του OpenWeatherMaps API στον κώδικα της εργασίας σας που θα ανεβεί 
δημόσια στο GitHub.

Θα πρέπει να βάλετε το API ΚΕΥ σε ένα αρχείο στον φάκελο /user/.gradle του υπολογιστή σας και να το διαβάζετε απο εκεί.

```
1. Προσθέστε μια γραμμή στο αρχείο ~/.gradle/gradle.properties  (αν δεν υπάρχει ήδη, δημιουργήστε το), η γραμμή θα πρέπει να έχει τη μορφή:
MyOpenWeatherMapApiKey=”YOUR_API_KEY”

2. Στον κωδικά σας, αλλάξτε το αρχείο app/build.gradle 

android {

....
....
...
    buildTypes.each {
        it.buildConfigField 'String', 'OPEN_WEATHER_MAP_API_KEY', MyOpenWeatherMapApiKey
    }
}
```
