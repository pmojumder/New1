✅ 2-3. Personal Message (personal_message.py)
python
Copy
Edit
# 2-3 Personal Message
name = "Eric"
print("Hello " + name + ", would you like to learn some Python today?")
✅ 2-4. Name Cases (name_cases.py)
python
Copy
Edit
# 2-4 Name Cases
name = "ada lovelace"

print(name.lower())     # lowercase
print(name.upper())     # UPPERCASE
print(name.title())     # Title Case
✅ 2-5. Famous Quote (famous_quote.py)
python
Copy
Edit
# 2-5 Famous Quote
print('Albert Einstein once said, "A person who never made a mistake never tried anything new."')
✅ 2-6. Famous Quote 2 (famous_quote_2.py)
python
Copy
Edit
# 2-6 Famous Quote with variables
famous_person = "Albert Einstein"
message = famous_person + ' once said, "A person who never made a mistake never tried anything new."'
print(message)
✅ 2-7. Stripping Names (stripping_names.py)
python
Copy
Edit
# 2-7 Stripping Names
name_with_spaces = "\t\n  Alice Smith  \n\t"

print("Original with whitespace:")
print(name_with_spaces)

print("Using lstrip():")
print(name_with_spaces.lstrip())

print("Using rstrip():")
print(name_with_spaces.rstrip())

print("Using strip():")
print(name_with_spaces.strip())
