# Writing in the household's languages

Aaj Kya Bana uses six locales. Two people read what you write, and each may use a different one.

| locale | language | script | respect form for the cook |
|---|---|---|---|
| `en` | English | Latin | the name alone: "Sunita," |
| `hi` | Hindi | Devanagari | "सुनीता जी", आप-form: "भिगो दीजिए" |
| `hi-Latn` | Hinglish | Latin | "Sunita ji", aap-form: "le lijiye", "bhigo dena" |
| `mr` | Marathi | Devanagari | "सुनीता ताई", तुम्ही-form: "भिजत घाला" |
| `te` | Telugu | Telugu | "లక్ష్మి గారు", మీరు-form: "నానబెట్టండి" |
| `kn` | Kannada | Kannada | "ಮಂಜುಳಾ ಅವರೇ", ನೀವು-form: "ನೆನೆಸಿ" |

## Which language

- **Cook card**: the helper's `language` from `get_household`. Never the app's language. A Telugu family with a cook from Bihar sends the card in Hinglish.
- **Everything the person reads**: `log_activity` summaries, `Cart.note`, `fail_task` reasons and your chat replies. Use the person's app locale, the `locale` that `get_household` returns. When it is missing, use the language the person writes to you in, or `en` in an absent run.

Product names stay in Latin letters in every locale: Instamart, Zepto, Zomato, Swiggy, WhatsApp. Numbers use Latin digits and Indian grouping (₹1,43,300). Times are the cook's visit times as people say them: "7:30".

## The menu card

- Greet the cook by name with the respect form above. Then say the day and how many people eat.
- One line per meal. Slot, then the dishes. Leave out slots the cook doesn't cover.
- Then one or two short sentences on what is already done or coming: what's soaking, what's in the fridge, what arrives before they come.
- Short sentences and everyday words. Many cooks read slowly, and the phone may read the card aloud as a voice note.
- Write dish names the way homes in that language say them, in that language's script: "पोहे" in Marathi, "ಅವಲಕ್ಕಿ" in Kannada, "rajma chawal" in Hinglish. Never translate a dish into English.
- Hinglish is Hindi in Latin letters, spelled the way people type on WhatsApp: "kal", "mein", "hai", "dena". Keep English words cooks already use: lunch, fridge, gas, cooker.
- Limits: 6 lines, 60 words, 400 characters. Telugu and Kannada words run long, so the character limit usually binds first.
- No emoji and no exclamation marks.

## One example per language

All six say the same thing: tomorrow, 4 people, three meals, one prep note, one fridge note. The Telugu and Kannada cards use dishes those homes cook.

`en`
```
Sunita, food for tomorrow, 4 people
Breakfast: poha, chai
Lunch: rajma chawal, kachumber
Dinner: phulka, palak paneer
Please soak the rajma tonight. Paneer is in the fridge.
```

`hi`
```
सुनीता जी, कल का खाना, 4 लोग
नाश्ता: पोहा, चाय
दोपहर: राजमा चावल, कचूमर
रात: फुल्का, पालक पनीर
राजमा आज रात भिगो दीजिए। पनीर फ्रिज में है।
```

`hi-Latn`
```
Sunita ji, kal ka khana, 4 log
Nashta: poha, chai
Lunch: rajma chawal, kachumber
Raat: phulka, palak paneer
Rajma aaj raat bhigo dena. Paneer fridge mein hai.
```

`mr`
```
सुनीता ताई, उद्याचं जेवण, 4 जण
नाश्ता: पोहे, चहा
दुपार: राजमा भात, कोशिंबीर
रात्री: फुलके, पालक पनीर
राजमा आज रात्री भिजत घाला. पनीर फ्रिजमध्ये आहे.
```

`te`
```
లక్ష్మి గారు, రేపటి వంట, 4 మంది
టిఫిన్: పెసరట్టు, అల్లం చట్నీ
మధ్యాహ్నం: పప్పు, అన్నం, బెండకాయ వేపుడు
రాత్రి: చపాతీ, పాలక్ పనీర్
పెసలు ఈ రాత్రి నానబెట్టండి. పనీర్ ఫ్రిజ్‌లో ఉంది.
```

`kn`
```
ಮಂಜುಳಾ ಅವರೇ, ನಾಳೆ ಅಡುಗೆ, 4 ಜನ
ತಿಂಡಿ: ಅವಲಕ್ಕಿ, ಕಾಫಿ
ಮಧ್ಯಾಹ್ನ: ಅನ್ನ, ಸಾರು, ಬೀನ್ಸ್ ಪಲ್ಯ
ರಾತ್ರಿ: ಚಪಾತಿ, ಪಾಲಕ್ ಪನೀರ್
ಮೊಸರು ಫ್ರಿಡ್ಜ್‌ನಲ್ಲಿದೆ. ತರಕಾರಿ 7:30ಕ್ಕೆ ಬರುತ್ತೆ.
```

## Other cook messages

You write only the menu card. The backend writes the `change`, `off_day` and `groceries` messages from the `cook.*` templates in `i18n/strings/<locale>.json`. Use the same register as those templates so the cook gets one voice from the household.

## A male cook

Each helper from `get_household` has `gender`, `"female"` unless the household set `"male"`. The examples above are for a woman. When `gender` is `male`:

- Hindi and Hinglish verbs that refer to the cook take the masculine form: "आएंगे", "aayenge", not "आएंगी", "aayengi". Imperatives such as "bhigo dena" and "ले लीजिए" stay the same.
- Marathi calls him "दादा" instead of "ताई". Marathi तुम्ही-form verbs already read the same for a man or a woman.
- Telugu "గారు", Kannada "ಅವರೇ" and English don't change.

The backend's `cook.*` templates follow the same rule through their `.m` keys, so match them.

`hi-Latn`, male cook
```
Ramesh ji, kal ka khana, 4 log
Nashta: poha, chai
Lunch: rajma chawal, kachumber
Raat: phulka, palak paneer
Rajma aaj raat bhigo dena. Aap aayenge tab tak paneer aa jayega.
```

`mr`, male cook
```
रमेश दादा, उद्याचं जेवण, 4 जण
नाश्ता: पोहे, चहा
दुपार: राजमा भात, कोशिंबीर
रात्री: फुलके, पालक पनीर
राजमा आज रात्री भिजत घाला. पनीर फ्रिजमध्ये आहे.
```
