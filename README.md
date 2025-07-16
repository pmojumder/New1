✅ 3-4. Guest List
python
Copy
Edit
# Exercise 3-4: Guest List

guest_list = ["Albert Einstein", "Maya Angelou", "Leonardo da Vinci"]

# Send invitation messages
print("Dear " + guest_list[0] + ", would you like to join me for dinner?")
print("Dear " + guest_list[1] + ", would you like to join me for dinner?")
print("Dear " + guest_list[2] + ", would you like to join me for dinner?")
✅ 3-5. Changing Guest List
python
Copy
Edit
# Exercise 3-5: One guest can't make it

# Original guest list
guest_list = ["Albert Einstein", "Maya Angelou", "Leonardo da Vinci"]

# One guest can't make it
print(guest_list[1] + " can't make it to the dinner.")

# Replace the guest
guest_list[1] = "Marie Curie"

# Send new invitations
print("Dear " + guest_list[0] + ", would you like to join me for dinner?")
print("Dear " + guest_list[1] + ", would you like to join me for dinner?")
print("Dear " + guest_list[2] + ", would you like to join me for dinner?")
✅ 3-6. More Guests
python
Copy
Edit
# Exercise 3-6: Found a bigger dinner table

# Start from updated guest list from 3-5
guest_list = ["Albert Einstein", "Marie Curie", "Leonardo da Vinci"]

print("Good news! I found a bigger dinner table.")

# Add new guests
guest_list.insert(0, "Nikola Tesla")               # Beginning
guest_list.insert(2, "Ada Lovelace")               # Middle
guest_list.append("Stephen Hawking")               # End

# Send updated invitations
for guest in guest_list:
    print("Dear " + guest + ", would you like to join me for dinner?")
