#![feature(if_let)]
#![feature(while_let)]

extern crate ncurses;
extern crate alloc;


mod deku {
    use alloc::heap::{
        EMPTY,
        allocate,
        reallocate,
        deallocate
    };

    /// Implements a Gap Buffer. Used to represent the visible portion
    /// of code in a file with a large gap in it for efficient editing
    /// of large files.
	pub struct Buffer {
        pub gap:  (*mut u8, *mut u8),
        pub data: *mut u8,
        pub size: uint
    }

    impl Buffer {
        /// Create a new Gap Buffer based on an existing buffer (such as
        /// from a file), and allocates new space for that buffer with
        /// the addition of a gap for modification.
        fn new(buffer: &[u8], gap_size: uint) -> Buffer {
            unsafe {
                let data = allocate(buffer.len() + gap_size, 1);

                Buffer {
                    gap:  (data.offset(1), data.offset(1 + gap_size as int)),
                    data: data,
                    size: buffer.len() + gap_size
                }
            }
        }
    }

    impl Drop for Buffer {
        /// Don't leak! Clean up the Gap Buffer.
        fn drop(&mut self) {
            unsafe {
                if !self.data.is_null() {
                    deallocate(self.data, self.size, 1);
                }
            }
        }
    }
    
    /// A Window represents a block of screenspace that contains a gap
    /// buffer representing the contents. Gap buffers may only contain
    /// part of a file at any point, but they represent the contents of
    /// the window. Even if not file backed. Size is used to curb the
    /// edges of the window and allows rendering line numbers and things
    /// like that.
    pub struct Window {
        pub buffer: Option<Buffer>,
        pub size:   [i32, ..2],
        pub view:   String
    }
}


mod ui {
    use ncurses;

    /// Represents the state of the UI for the program, contains the
    /// main window, which contains a default buffer by default.
	pub struct UIState {
        pub window: super::deku::Window
    }

    impl UIState {
        /// Initialize the UI.
        /// TODO: Make this work for more than just ncurses, like X.
        pub fn init() -> Option<UIState> {
            ncurses::initscr();
            ncurses::noecho();
            ncurses::start_color();
            ncurses::use_default_colors();
            ncurses::keypad(ncurses::stdscr, true);

            let mut cx = 0;
            let mut cy = 0;

            ncurses::getmaxyx(ncurses::stdscr, &mut cy, &mut cx);

            Some(UIState {
                window: super::deku::Window {
                    buffer: None,
                    size:   [cx, cy],
                    view:   String::new()
                }
            })
        }

        /// Fetch input from the user, and switch the mode of the UI
        /// depending on what input the user has entered. This way,
        /// we can seperate input handling from UI rendering.
		pub fn input(&mut self) {
            let character = ncurses::getch();

            if let Some(character) = ::std::char::from_u32(character as u32) {
                self.window.view.push(character);
            }
        }
        
        /// Render the UI, things might change depending on events
        /// occuring in the input call.
        pub fn draw(&self) {
            ncurses::clear();
            ncurses::mvprintw(0, 0, self.window.view.as_slice());
        }
    }
}


pub fn main() {
    use ui::UIState;

    if let Some(mut state) = UIState::init() {
        loop {
            state.input();
            state.draw();
        }
    }
}
