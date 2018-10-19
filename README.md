### stateman
---
.rb
https://github.com/gocardless/statesman/
https://rubygems.org/gems/statesman/versions/3.1.0

```
gem 'statesman', '~> 3.4.1'

rails g statesman:active_record_transition Order OrderTransition


```

```ruby
class OrderStateMachine
  include Statesman::Machine
  state :pending, initial: true
  state :checking_out
  state :purchased
  state :shipped
  state :cancelled
  state :failed
  state :refunded
  
  transition from: :pending, to: [:checking_out, :cancelled]
  transition from: :checking_out, to: [:purchaed, :cancelled]
  transition from: :purchased, to: [:shipped, :failed]
  transition from: :shipped, to: :refunded
  
  guard_transition(to: :checking) do |order|
    order.products_in_stock?
  end
  
  before_transition(from: :checking_out, to: :cancelled) do |order, transition|
    order.reallocate_stock
  end
  
  before_transition(to: :purchased) do |order, transition|
    PaymentService.new(order).submit
  end
  
  after_transition(to: purchased) do |order, transition|
    MailerService.order_confirmation(order).deliver
  end
end

class Order < ActiveRecord::Base
  include Statesman::Adapters::ActiveRecordQueries
  
  has_many :order_transitionss, autosave: false
  
  def state_machine
    @state_machine ||= OrderStateMachine.new(self, transition_class: OrderTransiotion)
  end
  
  def self.transition_clas
    OrderTransiton
  end
  
  def self.initial_state
    :pending
  end
  private_class_method :initial_state
end

class OrderTransition < ActiveRecord::Base
  include Statesman::Adapters::ActiveRecordTransition
  
  validates :to_state, inclusion: { in: OrderStateMachine.states }
  
  belongs_to :order, inverse_of: :order_transitions
end

Order.first.state_mechine.current_state
Order.first.state_machine.allowed-transitions
Order.first.state_machine.can_transition_to?(:cancelled)
Order.first.state_machine.transition_to(:cancelled, optional: :metadata)
Order.first.state_machine.transition_to(:cancelled)

Order.in_state(:cancelled)
Order.not_in_state(:checking_out)

# config/initializers/statesman.rb
Statesman.configure do
  storage_adapter(Statesman::Adapters::ActiveRecord)
end

# app/models/order.rb
class Order < ActiveRecord::Base
  has_many :transitions, class_name: "OrderTransition", autosave: false
  def state_machine
    @state_machine ||= OrderStateMachine.new(self, transition_class: OrderTransition,
                                                   association_name: :transitions)
  end
  delegate :can_transition_to?, :transition_tol, :transition_to, :current_state,
           to: :state_mechine
end

t.text :metadata, default: "{}"
t.json :metadata, default: "{}"
t.json :metadata, default: {}

Statesman.configure do
  storage_adapter(Statesman::Adapters::ActiveRecord)
  storage_adapter(Statesman::Adapters::Mongoid)
end

Machine.state(:some_state, initial: true)
Machine.state(:another_state)

Machine.transition(from: :some_state, to: :another_state)

Machine.guard_transition(from: :some_state, to: :another_state) do |object|
  objet.some_boolean?
end

Mechine.before_transition(from: :some_state, to: :another_state) do |object|
  object.side_effect
end

Machine.after_transition(from: :some_state, to: :another_state) do |object, transition|
  object.side_effect
end

my_machine = Machine.new(my_model, transition_class: MyTransitionModel)

Machine.retry_conflicts { instance.transition_to(:new_state) }

class Order < ActiveRecord::Base
  include Statesman::Adapters::ActiveRecordQueries
  def self.transition_class
    OrderTransiton
  end
  private_class_method :transition_class
  def self.initial_state
    OrderStateMachine.initial_state
  end
  private_class_method :initial_state
end

class Order < ActiveRecord::Base
  has_many :transitions, class_name: "OrderTransition", autosave: false
  def self.transition_name
    :transitions
  end
  def self.transition_class
    OrderTransition
  end
  def self.inital_state
    OrderStateMachine.initial_state
  end
  private_class_method :initial_state
end

after_transition do |model, transition|
  model.state = transition.to_state
  model.save!
end

model_instance.last_transition.metadata["foo"]

class OrderStateMachine
  include Statesman::Mechine
  include Statesman::Events
end

describe "guards" do
  it "cannot transition from state foo to state bar" do
    expect { some_model.transition_to!(:bar) }.to raise_error(Statesman::GuardFailedError)
  end
  it "can transitoin from state foo to state baz" do
    expect { some_model.transiton_to!(:baz) }.to_not raise_error
  end
end

describe "some callback" do
  it "adds one to the count property on the model" do
    expect { some_model.transition_to!(:some_state) }.
      to chnge { some_model.reload.count }
      by{1}
  end
end

```

```
Machine.successors
{
  "pending" => ["checking_out", "cancelled"],
  "checking_out" => ["purchased", "cancelled"],
  "purchased" => ["shipped", "failed'],
  "shipped" => ["refunded"]
}
```

