# Aviator

A lightweight library for communicating with the OpenStack API.

## Installation

Add this line to your application's Gemfile:

    gem 'aviator'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install aviator

## Usage

```ruby
require 'aviator'

# Authenticate against openstack as specified in :config_file
# generating a new default token in the process.
session = Aviator::Session.new(
            config_file: 'path/to/aviator.yml',
            env:         :production,
            log_file:    'path/to/aviator.log'
          )

# Authenticate against openstack as specified in :config_file
# but log in as specified user, generating a new default token
#in the process
session = Aviator::Session.new(
            config_file: 'path/to/aviator.yml', 
            env:         :production,
            username:     username, 
            password:     password, 
            log_file:    'path/to/aviator.log'
          )

# In both methods above, Aviator::Session automatically caches the connection
# details in a central store such as mysql or redis (must be specefied in :config_file)

# Get the session id for later retrieval
session_id = session.id

# Restore a previous session. DOES NOT generate a new token unless token is expired
session = Aviator::Session.new(
            connection_id:  connection_id, 
            log_file:      'path/to/redstack.log'
          )

# Return a new session object scoped to the default token
scoped_session = session.scope(:default)

# Return a new session object with a token scoped to :tenant_id. If
# scoped token does not exist, create one. If scoped token exists but is
# expired, create a new one. Otherwise, re-use existing one
scoped_session = session.scope(tenant_id: '...')

# Return a new session object with a token scoped to :tenant_name. If
# scoped token does not exist, create one. If scoped token exists but is
# expired, create a new one. Otherwise, re-use existing one
scoped_session = session.scope(tenant_name: '...')

# Get connection to Keystone
keystone = scoped_session.identity_service

# Create a new tenant
response = keystone.request(:create_tenant) do |params|
  params[:name]        = 'Project'
  params[:description] = 'My Project'
  params[:enabled]     = true
end

# Note that the name of the params will always match the name 
# in the official OpenStack API docs. You can also reference a
# param above as `params['paramname']` or `params.paramname`

# You may also query Aviator for the parameters via the CLI.
# With the Aviator gem installed, run the following:

aviator describe openstack identity create_tenant

# Be explicit about API version and endpoint type to use. Note that if Aviator does not
# implement the given version + endpoint, an exception will be raised. If Aviator
# implements it but the the underlying OpenStack service does not, Aviator will not
# raise an error and will return a Response object with the appropriate status and body
keystone_v3 = keystone.use(api_version: 'v3', endpoint_type: 'admin')

response = keystone_v3.request(:list_projects) do |params|
  params['project_id'] = project_id
end


keystone_v2 = keystone.use(api_version: 'v2', endpoint_type: 'admin')

response = keystone_v2.request(:create_tenant) do |params|
  params['name']        = 'ACME corp'
  params['description'] = 'A description...'
  params['enabled']     = true
end
```
  
## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
