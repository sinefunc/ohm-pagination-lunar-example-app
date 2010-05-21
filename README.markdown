A Short Demo of Ohm + Pagination + Lunar formula
================================================

    require 'rubygems'
    require 'sinatra'
    require 'ohm'
    require 'lunar'
    require 'pagination'
    require 'haml'

    helpers Pagination::Helpers

    class Post < Ohm::Model
      attribute :body
      attribute :tags
      attribute :votes

      def create
        if super
          index 
          return true
        end
      end

    protected
      def index
        Lunar.index Post do |i|
          i.id(id)
          i.text :body, body
          i.text :tags, tags

          i.sortable :votes, votes
        end
      end
    end

    get '/' do
      @posts = paginate Post.all, 
        :per_page => 10, :page => params[:page],
        :order => "DESC"
      
      haml :home
    end

    post '/post' do
      Post.create(:body => params[:body], :tags => params[:tags])
      redirect '/'
    end

    get '/search' do
      results = Lunar.search(Post, :q => params[:q], :tags => params[:tags])

      @posts = paginate results, :page => params[:page], :per_page => 10,
        :sort_by => :votes, :order => "DESC"

      haml :home
    end

    if Post.all.size == 0
      Post.create(:body => "The all new macbook pro 17inch", :tags => "apple macbook pro 17", :votes => 20)
      Post.create(:body => "apple iPad 3G", :tags => "apple ipad 3G tablet", :votes => 11)
      Post.create(:body => "apple iPad", :tags => "apple ipad tablet", :votes => 2)
      Post.create(:body => "apple iPhone 3GS", :tags => "apple iphone mobile", :votes => 5)
      Post.create(:body => "apple iPhone 3G", :tags => "apple iphone mobile", :votes => 10)
    end

    __END__
    @@ home
    %h1 All Posts (#{@posts.total})

    %form(action='/search' method='get')
      %fieldset
        %legend Search Posts
        %label
          %span Keywords:
          %input(type='text' name='q')

        %label
          %span Tags:
          %input(type='text' name='tags')
          
        %button(type='submit') Go

    %ul
      - @posts.each do |post|
        %li
          %p= post.body
          %p 
            Tagged: #{post.tags}
            |
            Votes: #{post.votes}

    %form(action='/post' method='post')
      %fieldset
        %label(for='body')
          %span Body
          %br
          %textarea(name='body' cols=50 rows=5)
        
        %br

        %label(for='tags')
          %span Tags
          %br
          %input(type='text' name='tags')
        
      %fieldset
        %button(type='submit') Post!

    = pagination @posts
