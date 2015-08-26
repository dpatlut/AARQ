
# AARQ aka ( Advanced ActiveRecord Querying )

### What is active record you ask?

#### Active record is kind of like "Middlewear" that interacts with the Model and the Database.

Going back to miles analogy:

> View: "Hey, controller, the user just told me he wants item 4 deleted."

> Controller: "Hmm, having checked his credentials, he is allowed to do that... Hey, model, I want you to get item 4 and do whatever you do to delete it."

> Model: "Item 4... got it. It's deleted. Back to you, Controller."

**** MODEL USES ACTIVE RECORD HERE TO DO THE DELETION ****

> Controller: "Here, I'll collect the new set of data. Back to you, view."

> View: "Cool, I'll show the new set to the user now."


___

### Why do we use it?
1. It is wayyyyyyy faster int terms of how much you type...
	example :

		Active Record : client = Client.find(10)
		SQL Equivilant : SELECT * FROM clients WHERE (clients.id = 10) LIMIT 1
		
	Which one would you rather type out?
	Exactly.
2.  In Active Record, objects carry both persistent data and behavior which operates on that 	 data. Active Record takes the opinion that ensuring data access logic as part of the 		object will educate users of that object on how to write to and read from the database.

___

Active Record as a ORM ( Object Relational Mappinging)

	Active Record gives us several mechanisms
		1.Represent models and their data.
		2.Represent associations between these models.
		3.Represent inheritance hierarchies through related models.
		4.Validate models before they get persisted to the database.
		5.Perform database operations in an object-oriented fashion.

___
### So now to the practical stuff...


When we create a model we link it to active record which is essentially linked to our databsae like so :

	class Product < ActiveRecord::Base
	end

and a table :

	CREATE TABLE products (
   		id int(11) NOT NULL auto_increment,
   		name varchar(255),
   		PRIMARY KEY  (id)
		);


***class name convention is capital where as tables are lowercase.***

___
### Now to querys and advanced stuff.. mostly for finding multiple things across multiple tables

We all know the basic Crud stuff
	ex. .create , .update , .all , .destroy , .find  etc.

1.Retrieving Single Objects ( one thing at a time man! )

    client = Client.find(10)
    .first (!)
    .last (!)

2.Retrieving Multiple Objects ( I need like 10 of those )

    client = Client.find([1, 10])

	        a)Batches

            	User.all.each do |user|
            	  NewsLetter.weekly_deliver(user)
            	end
	
            	User.find_each do |user|
            	  NewsLetter.weekly_deliver(user)
            	end

            	Invoice.find_in_batches(:include => :invoice_lines) do |invoices|
              		export.add_invoices(invoices)
            	end

                		--Options

                        		User.find_each(:batch_size => 5000) do |user|
                          			NewsLetter.weekly_deliver(user)
                        		end
                        
                        		User.find_each(:start => 2000, :batch_size => 5000) do |user|
                          			NewsLetter.weekly_deliver(user)
                        		end
                        
                        		Batch Size => Which primary id you want to stop at
                        		Start => Which primary id to start on 

3. Conditions ( If Jason is.... where... )
   >When we need to specify a certain condition (.where)
		
    a) Array
		
        Client.where("orders_count = ?", params[:orders])
        		Client.where("orders_count = ? AND locked = ?", params[:orders], false)
        			1.Range
        				Client.where(:created_at => (params[:start_date].to_date)..(params[:end_date].to_date))
        			2.Placeholders
        				Client.where("created_at >= :start_date AND created_at <= :end_date",
          				{:start_date => params[:start_date], :end_date => params[:end_date]})

	b) Hashes
	
		Client.where(:locked => true)
		Client.where('locked' => true)
		Client.where(:created_at => (Time.now.midnight - 1.day)..Time.now.midnight)
		Client.where(:orders_count => [1,3,5])

	Adding specific arguments? No problemo

    	User.where(:email => "foo@bar.com")
    	User.where("email" => "foo@bar.com")
    	User.where("email = 'foo@bar.com'")
    	User.where("email = ?", "foo@bar.com")
    	User.where("users.email" => "foo@bar.com")

	There are multiple ways to use these methods :
	
        # via a model
        Post.any?
        Post.many?
    
        # via a relation
        Post.where(:published => true).any?
        Post.where(:published => true).many?
    
        # via an association
        Post.first.categories.any?
        Post.first.categories.many?

4. Ordering ( your room is so messy! )
   >When we need to order our content specifically

    	Client.order("created_at")
    	Client.order("created_at DESC")
    	Client.order("created_at ASC")
    	Client.order("orders_count ASC, created_at DESC")

5. Selecting Specific Fields ( I need.... )
    >When we need a specific field 

        Client.select("viewable_by, locked")
        Client.select(:name).uniq

6. Limits ( No more cookies for you! )
   >When we need to limit how much specifically 

    	Client.limit(5)
    	Client.limit(5).offset(30)

7. Joining ( Mashup! )
   >When we need to find stuff in multiple tables
	
    	ex.
    	 class Client < ActiveRecord::Base
          has_one :address
          has_one :mailing_address
          has_many :orders
          has_and_belongs_to_many :roles
        end
        class Address < ActiveRecord::Base
          belongs_to :client
        end
        class Role < ActiveRecord::Base
          has_and_belongs_to_many :clients
        end


    	User.joins(:posts)
    	=> SELECT "users".* FROM "users" INNER JOIN "posts" ON "posts"."user_id" = "users"."id"
    	
    	or
    
    	User.joins("LEFT JOIN bookmarks ON bookmarks.bookmarkable_type = 'Post' AND bookmarks.user_id = users.id")
    	=> SELECT "users".* FROM "users" LEFT JOIN bookmarks ON bookmarks.bookmarkable_type = 'Post' AND   bookmarks.user_id = users.id

    	Post.joins(:category, :comments)
    	Post.joins(:comments => :guest)

8. Eager Loading ( Load , Load , Load your db )
	
    	clients = Client.limit(10)
     
    	clients.each do |client|
      		puts client.address.postcode
    	end

	This code looks fine at the first sight. But the problem lies within the total number of     queries executed. The above code executes 1 ( to find 10 clients ) + 10 ( one per each client to load the address ) = 11 queries in total.
	
 	 Solution : Active Record got yo back
 	 
	 2 queries vs 11 queries


    	clients = Client.includes(:address).limit(10)
    	clients.each do |client|
    	  puts client.address.postcode
    	end

	    Multiple : 
    	Post.includes(:category, :comments)
    	Category.includes(:posts => [{:comments => :guest}, :tags]).find(1)

9. Pluck ( pluck you! )

    	Client.where(:active => true).pluck(:id)
    	# SELECT id FROM clients WHERE active = 1
     
    	Client.uniq.pluck(:role)
    	# SELECT DISTINCT role FROM clients
    
    	Client.pluck(:id)

10. Existance ( Do we really exist? )

    	Client.exists?(1)
    	Client.exists?(1,2,3)
    	Client.where(:first_name => 'Ryan').exists?

11. Calculations ( Math yo! )

    	Client.count
    	Client.where(:first_name => 'Ryan').count
    	Client.includes("orders").where(:first_name => 'Ryan', :orders => {:status =>        'received'}).count
    	Client.average("orders_count")
    	Client.minimum("age")
    	Client.maximum("age")
    	Client.sum("orders_count")



    Guide : http://guides.rubyonrails.org/v2.3.11/active_record_querying.html











