#Ref : https://reinink.ca/articles/calculating-totals-in-laravel-using-conditional-aggregates

For the purpose of this article, let's say that we have a subscribers database table with data in this format:

<table class="w-full">
    <tbody><tr>
        <th class="p-1 text-left">name</th>
        <th class="p-1 text-left">email</th>
        <th class="p-1 text-left">status</th>
    </tr>
    <tr>
        <td class="p-1 border-t whitespace-no-wrap">Adam Campbell</td>
        <td class="p-1 border-t whitespace-no-wrap">adam@hotmeteor.com</td>
        <td class="p-1 border-t whitespace-no-wrap">confirmed</td>
    </tr>
    <tr>
        <td class="p-1 border-t whitespace-no-wrap">Taylor Otwell</td>
        <td class="p-1 border-t whitespace-no-wrap">taylor@laravel.com</td>
        <td class="p-1 border-t whitespace-no-wrap">unconfirmed</td>
    </tr>
    <tr>
        <td class="p-1 border-t whitespace-no-wrap">Jonathan Reinink</td>
        <td class="p-1 border-t whitespace-no-wrap">jonathan@reinink.ca</td>
        <td class="p-1 border-t whitespace-no-wrap">cancelled</td>
    </tr>
    <tr>
        <td class="p-1 border-t whitespace-no-wrap">Adam Wathan</td>
        <td class="p-1 border-t whitespace-no-wrap">adam.wathan@gmail.com</td>
        <td class="p-1 border-t whitespace-no-wrap">bounced</td>
    </tr>
</tbody></table>

Case 1 : 

	$total = Subscriber::count();
	$confirmed = Subscriber::where('status', 'confirmed')->count();
	$unconfirmed = Subscriber::where('status', 'unconfirmed')->count();
	$cancelled = Subscriber::where('status', 'cancelled')->count();
	$bounced = Subscriber::where('status', 'bounced')->count();


Case 2 : 
	
	$subscribers = Subscriber::all();
	$total = $subscribers->count();
	$confirmed = $subscribers->where('status', 'confirmed')->count();
	$unconfirmed = $subscribers->where('status', 'unconfirmed')->count();
	$cancelled = $subscribers->where('status', 'cancelled')->count();
	$bounced = $subscribers->where('status', 'bounced')->count();

	Explan : Here we're making a single database query to get all the subscribers, and then running counts on the resulting collection. The thing is, this approach is actually significantly worse than the multiple queries solution. If our application has thousands or millions of subscribers, the time to process all the records will be slow, and will use a ton of memory.


Case 3 : Conditional aggregates
	
	select
	  count(*) as total,
	  count(case when status = 'confirmed' then 1 end) as confirmed,
	  count(case when status = 'unconfirmed' then 1 end) as unconfirmed,
	  count(case when status = 'cancelled' then 1 end) as cancelled,
	  count(case when status = 'bounced' then 1 end) as bounced
	from subscribers


	using query builder -

		$totals = DB::table('subscribers')
	    ->selectRaw('count(*) as total')
	    ->selectRaw("count(case when status = 'confirmed' then 1 end) as confirmed")
	    ->selectRaw("count(case when status = 'unconfirmed' then 1 end) as unconfirmed")
	    ->selectRaw("count(case when status = 'cancelled' then 1 end) as cancelled")
	    ->selectRaw("count(case when status = 'bounced' then 1 end) as bounced")
	    ->first();


    On Boolean Columns - 


	    $totals = DB::table('subscribers')
	    ->selectRaw('count(*) as total')
	    ->selectRaw('count(is_admin or null) as admins')
	    ->selectRaw('count(is_treasurer or null) as treasurers')
	    ->selectRaw('count(is_editor or null) as editors')
	    ->selectRaw('count(is_manager or null) as managers')
	    ->first();


    Explanation : This works since the count aggregate ignores null columns. Unlike in PHP where false or null returns false, in SQL (and JavaScript for that matter) it returns null. Basically, A or B returns the value A if A can be coerced into true; otherwise, it returns B.


    Using Filter Clause with Postgres - 

    	$totals = DB::table('subscribers')
	    ->selectRaw('count(*) as total')
	    ->selectRaw('count(*) filter (where is_admin) as admins')
	    ->selectRaw('count(*) filter (where is_treasurer) as treasurers')
	    ->selectRaw('count(*) filter (where is_editor) as editors')
	    ->selectRaw('count(*) filter (where is_manager) as managers')
	    ->first();


Another Intresting Laravel link : 
https://reinink.ca/articles/dynamic-relationships-in-laravel-using-subqueries