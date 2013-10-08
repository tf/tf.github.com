---
title: Decorating Active Record Models
layout: post
extract: Use case oriented presentations
---

In the quest against god object active records, decorators provide a
nice way to group functionality according to features of your
application. They help to keep your models lean and ease separation of
responsibilities.

### A simplified View

Often certain parts of an application only need to act upon a
simplified projection of the actual domain model. Imagine for example
an `Entry` model which consists of various `Revisions`. Furthermore,
let's say, for each entry, there is always at most one published
revision:

    class Entry < ActiveRecord::Base
      has_many :revisions
      has_one :published_revision, -> { published }, class: 'Revision'
    end

    class Revision < ActiveRecord::Base
      scope :published, -> { where(published: true) }
    end

Only the contents of the published revision is supposed to be visible
on the public site. Instead of making our controllers and views deal
with the complexity of finding the right revision to display, we
introduce a decorator to handle the delegation.

    class PublishedEntry
      def initialize(entry)
        @entry = entry
        @published_revision = entry.published_revision
      end

      delegate :title, :description, :to => :@published_revision
    end

Code using the decorator can now remain totally unaware of the
revision concept.  Even better, once we decide to change the logic
determining the correct revision for public display, there is a single
place to make that change.

### A Place for Finders

Since our decorator class is tailored for a rather specific use case,
we know what data is going to be needed. This knowledge can be
captured in a custom finder which performs appropriate eager loading.

    class PublishedEntry
      // ...

      def self.find(id)
        new(Entry.find(id).includes(:published_entry));
      end
    end

No need to either scatter these details across controller actions, nor
let a bunch of unrelated finder methods pile up in the `Entry` model.

### Quacking like a Model

From the controller's point of view, the decorator looks a lot like
the `ActiveRecord` model.

    class EntriesController < ApplicationController
      def show
        @entry = PublishedEntry.find(params[:id])
      end
    end

To preserve Rail's magic when passing records to routing, render and
form helpers, we delegate the required `ActiveModel` methods to the
decorated `Entry`.

    class PublishedEntry
      include ActiveModel::Conversion
      extend ActiveModel::Naming

      delegate :to_model, :to_key, :persisted?, :to => :@entry

      def initialize(entry)
        @entry = entry
        @published_revision = entry.published_revision
      end

      // ...
    end

`PublishedEntry` now basically looks like `Entry` to `ActionPack`
helpers.

# A Word about Testing

Whether it makes sense to test presentational decorators in isolation,
really depends on the kind of logic you end up encapsulating in
them. The more tightly they are integrated with active record specific
constructs like scopes or associations, the more a black box approach
makes sense verifying that all parts are wired up correctly.
